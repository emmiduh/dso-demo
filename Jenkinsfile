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
	stage('Software Component Analysis') {
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

	stage('Static Application Security Testing') {
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
	
	stage('Generate SBOM') {
	  steps {
	    container('maven') {
	      sh 'mvn org.cyclonedx:cyclonedx-maven-plugin:makeAggregateBom'
	    }
	  }
	  post {
	    success {
	      dependencyTrackPublisher projectName: 'sample-spring-app', projectVersion: '0.0.1', artifact: 'target/bom.xml', autoCreateProjects: true, synchronous: true
	      archiveArtifacts allowEmptyArchive: true, artifacts: 'target/bom.xml', fingerprint: true, onlyIfSuccessful: true
	    }
	  }
	}
	
	stage('OSS License Checker') {
	  steps {
	    container('licensefinder') {
	      sh '''
		 export LICENSE_FINDER_IGNORE_MAVEN_WRAPPER=true
		 gem install license_finder
		 license_finder
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
    stage('Package') {
      parallel {
        stage('Create Jarfile') {
          steps {
            container('maven') {
              sh 'mvn package -DskipTests'
            }
          }
        }
        stage('OCI Image BnP') {
          steps {
            container('kaniko') {
              sh '/kaniko/executor -f `pwd`/Dockerfile -c `pwd` --insecure --skip-tls-verify --cache=true --destination=docker.io/emmiduh93/dso-demo'
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
