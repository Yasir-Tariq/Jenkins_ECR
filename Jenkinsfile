pipeline {
  agent any
  parameters {
        string(name: 'family', defaultValue: 'app-db-cfn', description: 'Task Family for ecs.')
        string(name: 'ecs_cluster', defaultValue: 'mycluster', description: 'Enter the cluster name')
        string(name: 'service_name', defaultValue: 'sample-service', description: 'Enter service.')
           }
     stages {
        stage ('image push') {
            environment {
               token = sh(script: "eval aws ecr get-login --no-include-email --region us-east-2 | sed 's|https://||'", , returnStdout: true).trim()
           }
            steps {
                script {
                    withAWS(region:'us-east-2') {
                        sh "docker build -t tweet ."  
                        sh "docker tag tweet:latest 020046395185.dkr.ecr.us-east-2.amazonaws.com/tweet:${GIT_COMMIT}"
                        sh "${env.token}"
                        sh "docker push 020046395185.dkr.ecr.us-east-2.amazonaws.com/tweet:${GIT_COMMIT}"
                    }   
                }
              } 
        }
        stage ('ecs update'){
            steps {
                // script {
                withAWS(region:'us-east-2') {
                    sh '''ECR_IMAGE="020046395185.dkr.ecr.us-east-2.amazonaws.com/tweet:\${GIT_COMMIT}"
                        TASK_DEFINITION=$(aws ecs describe-task-definition --task-definition "${params.family}" --region "us-east-2")
                        NEW_TASK_DEFINTIION=$(echo \$TASK_DEFINITION | jq --arg IMAGE "\$ECR_IMAGE" \'.taskDefinition | .containerDefinitions[0].image = \$IMAGE | del(.taskDefinitionArn) | del(.revision) | del(.status) | del(.requiresAttributes) | del(.compatibilities)\')
                        NEW_TASK_INFO=$(aws ecs register-task-definition --region "us-east-2" --cli-input-json "\$NEW_TASK_DEFINTIION")
                        NEW_REVISION=$(echo \$NEW_TASK_INFO | jq \'.taskDefinition.revision\')
                        aws ecs update-service --cluster ${params.ecs_cluster} --service ${params.service_name} --task-definition ${params.family}:\${NEW_REVISION}
                '''
                }
                // }
                
            }
        }

    }
}




























