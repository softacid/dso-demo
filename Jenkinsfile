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
    stage('Test') {
      parallel {
        stage('Unit Tests') {
          steps {
            container('maven') {
              sh 'mvn test'
            }
          }
        }
      }
    }
    stage('SCA'){
      steps {
        container('maven') {
          catchError(buildResult: 'SUCCESS', stageResult: 'FAILURE') {
            sh 'mvn org.owasp:dependency-check-maven:check'
          }
        }
      }
      post {
        always {
            archiveArtifacts allowEmptyArchive: true, artifacts: 'target/dependency-check-report.html', fingerprint:true, onlyIfSuccessful: true
          //dependencyCheckPublisherpattern:'report.xml'
        }
      }
    }
    stage('GenerateSBOM') {
    steps {
        container('maven') {
            sh 'mvn org.cyclonedx:cyclonedx-maven-plugin:makeAggregateBom'
        }
    }
    post {
        success {
            dependencyTrackPublisher(
                projectName: 'sample-spring-app',
                projectVersion: '0.0.1',
                artifact: 'target/bom.xml',
                autoCreateProjects: true,
                synchronous: true
            )
            archiveArtifacts(
                allowEmptyArchive: true,
                artifacts: 'target/bom.xml',
                fingerprint: true,
                onlyIfSuccessful: true
            )
        }
    }
}

    stage('OSSLicenseChecker'){
      steps {
        container('licensefinder') {
          sh 'ls -al'
          sh '''#!/bin/bash --login
                 /bin/bash--login
                 rvm use default
                 gem install license_finder
                 license_finder
             '''
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
        stage('OCIImageBnP') {
           steps {
              container('kaniko') {

               sh '/kaniko/executor -f `pwd`/Dockerfile -c `pwd` --insecure --skip-tls-verify --cache=true --destination=docker.io/teodorb/dso-demo'
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
