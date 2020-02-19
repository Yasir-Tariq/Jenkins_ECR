pipeline {
  agent any
  parameters {
        string(name: 'family', defaultValue: 'app-db-cfn', description: 'Task Family for ecs.')
        string(name: 'ecs_cluster', defaultValue: 'mycluster222', description: 'Enter the cluster name')
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
            environment {
               ecr_image = "020046395185.dkr.ecr.us-east-2.amazonaws.com/tweet:${GIT_COMMIT}"
               task_definition = sh(script: "aws ecs describe-task-definition --task-definition ${params.family} --region 'us-east-2'")
               new_task_definition = sh(script: "echo ${env.task_definition} | jq  '.taskDefinition | .containerDefinitions[0].image = ${env.ecr_image} | del(.taskDefinitionArn) | del(.revision) | del(.status) | del(.requiresAttributes) | del(.compatibilities)'")
               new_task_info = sh(script: "aws ecs register-task-definition --region 'us-east-2' --cli-input-json ${env.new_task_definition}")
               new_revision = sh(script: "echo ${env.new_task_info} | jq '.taskDefinition.revision'")
           }
            steps {
                script {
                    withAWS(region:'us-east-2') {
                        sh "${env.ecr_image}"
                        sh "${env.task_definition}"
                        sh "${env.new_task_definition}"
                        sh "${env.new_task_info}"
                        sh "${env.new_revision}"
                        sh "aws ecs update-service --cluster ${params.ecs_cluster} --service ${params.service_name} --task-definition ${params.family}:${env.new_revision}"
                    }
                }
                
            }
        }

    }
}




























