pipeline {
  agent any
     stages {
         stage ('image build') {
            steps {
              script {
                docker.build('tweet')
              }
                }
            }
        stage ('image push') {
            steps {
              script {
                docker.withRegistry('020046395185.dkr.ecr.us-east-2.amazonaws.com'){
                  docker.image('tweet').push("${GIT_COMMIT}") //GIT_COMMIT is the environment variable containg the latest commit hash value from git
                }
              }
                }
            }
        }
    }