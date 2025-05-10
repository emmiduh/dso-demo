pipeline {
  agent {
    kubernetes {
      yamlFile 'build-agent.yaml'
      defaultContainer 'maven'
      idleMinutes 1
    }
  }
  stages {
    stage('Build') {
      parallel {
        stage('Compile') {
          steps {
            container('maven') {
              sh 'mvn compile'
            }
          }
        }
      }
    }
    stage('Static Analysis') {
      parallel {
	stage('SCA') {
	  steps {
	    container('maven') {
	      catchError(buildResult: 'SUCCESS', stageResult: 'FAILURE') {
		sh 'mvn org.owasp:dependency-check-maven:check'
	      }
	    }
	  }
	  post {
	    always {
	      archiveArtifacts allowEmptyArchive: true, artifacts: 'target/dependency-check-report.html', fingerprint: true, onlyIfSuccessful: true
    	      // dependencyCheckPublisher pattern: 'report.xml'
	    }
     	  }
	}
	stage('OSS License Checker') {
	  steps {
	    container('licensefinder') {
	      sh 'ls -la'
	      sh '''
		bash -lc "
		rvm use default &&
    		gem install license_finder &&
    		chmod +x ./mvnw &&
    		./mvnw org.codehaus.mojo:license-maven-plugin:download-licenses &&
    		license_finder"
		'''
	    }
	  }
	}
        stage('Unit Tests') {
          steps {
            container('maven') {
              sh 'mvn test'
            }
          }
        }
      }
    }
    
	stage('SAST') {
	  steps {
	    container('slscan') {
		sh 'scan --type java,depscan --build'
		}
	      }
	  post {
	    success {
	      archiveArtifacts allowEmptyArchive: true, artifacts: 'reports/*', fingerprint: true, onlyIfSuccessful: true
		}
	   }
	}	


    stage('Package') {
      parallel {
	stage('OCI Image BnP') {
          steps {
            container('kaniko'){
              sh '''/kaniko/executor -f $(pwd)/Dockerfile -c $(pwd) --insecure --skip-tls-verify --cache=true --destination=docker.io/emmiduh93/dso-demo'''
            }
          }
        }
        stage('Create Jarfile') {
          steps {
            container('maven') {
              sh 'mvn package -DskipTests'
            }
          }
        }
      }
    }

    stage('Deploy to Dev') {
      steps {
        // TODO
        sh "echo done"
      }
    }
  }
}
