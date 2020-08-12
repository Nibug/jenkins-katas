pipeline {
  agent any
  environment {
      docker_username = 'nibug18'
  }
  stages {
      stage('clone down') {
          steps{
            stash excludes: '.git', name: 'code'
          }
      }
      
    stage('Say Hello') {
      parallel {
        stage('Parallel execution') {
          steps {
            sh 'echo "hello world"'
          }
        }

        stage('build app') {
          agent {
            docker {
              image 'gradle:jdk11'
            }
          }
          options {
            skipDefaultCheckout(true)
          }
          steps {
            unstash 'code'
            sh 'ci/build-app.sh'
            archiveArtifacts 'app/build/libs/'
            stash excludes: '.git', name: 'code'
            sh label: '', script: 'ls'
            deleteDir()
            sh label: '', script: 'ls'
          }
        }
        stage('test app') {
          agent {
            docker {
              image 'gradle:jdk11'
            }
          }
          options {
            skipDefaultCheckout(true)
          }
          steps {
            unstash 'code'
            sh label: '', script: 'ci/unit-test-app.sh'
            junit 'app/build/test-results/test/TEST-*.xml'
          }
        }
      }
    }

    stage('push docker app') {
      when { not { branch 'dev/' } }
      options {
        skipDefaultCheckout(true)
      }

      environment {
        DOCKERCREDS = credentials('docker_login') //use the credentials just created in this stage
      }

      steps {
        unstash 'code'
        sh 'ci/build-docker.sh'
        sh 'echo "$DOCKERCREDS_PSW" | docker login -u "$DOCKERCREDS_USR" --password-stdin'
        sh 'ci/push-docker.sh'
      }
    }

      stage()
  } 
}