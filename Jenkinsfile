pipeline {
  agent any
     stages {
         stage ('image build') {
            steps {
            //   script {
            sh "docker build -t tweet ."
            //   }
                }
            }
        stage ('image push') {
            steps {
            //   script {
            // sh "aws ecr get-login --no-include-email --region us-east-2"
            sh "eval $(aws ecr get-login | sed 's|https://||')"
            sh "docker tag tweet:latest 020046395185.dkr.ecr.us-east-2.amazonaws.com/tweet:${GIT_COMMIT}"
            sh "docker push 020046395185.dkr.ecr.us-east-2.amazonaws.com/tweet:${GIT_COMMIT}"
                // }
              }
                }
            }
        }
