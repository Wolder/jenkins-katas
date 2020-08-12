pipeline {
  agent any
  environment {
    docker_username = "wolder"
  }
  stages {
    stage('Parallel execution') {
      parallel {
        stage('say hello') {
          steps {
            sh 'echo "hello world!"'
          }
        }

        stage('build app') {
          agent {
            docker {
              image 'gradle:jdk11'
            }
          }
          steps {
            sh 'ci/build-app.sh'
            archiveArtifacts 'app/build/libs/'
            sh 'ls'
            deleteDir()
            sh 'ls'
          }
        }
      }
    }
    stage('push docker app') {
      environment {
         DOCKERCREDS = credentials('docker_login') //use the credentials just created in this stage
      }

      steps {
	 unstash 'code' //unstash the repository code
	 sh 'ci/build-docker.sh'
	 sh 'echo "$DOCKERCREDS_PSW" | docker login -u "$DOCKERCREDS_USR" --password-stdin' //login to docker hub with the credentials above
	 sh 'ci/push-docker.sh'
      }
   }
}
}
