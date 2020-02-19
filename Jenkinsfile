pipeline {
  agent any
  parameters {
        string(name: 'family', defaultValue: 'app-db-cfn', description: 'Task Family for ecs.')
        string(name: 'ecs_cluster', defaultValue: 'mycluster222', description: 'Enter the cluster name')
        string(name: 'service_name', defaultValue: 'sample-service', description: 'Enter service.')
           }
     stages {
        stage ('image push') {
            steps {
                script {
                    withAWS(region:'us-east-2') {
                        sh "aws ecr get-login --no-include-email --region us-east-2"
                        sh "docker build -t tweet ."
                        sh "docker tag tweet:latest 020046395185.dkr.ecr.us-east-2.amazonaws.com/tweet:${GIT_COMMIT}"
                        sh "docker push 020046395185.dkr.ecr.us-east-2.amazonaws.com/tweet:${GIT_COMMIT}"
                    }
                }
              } 
        }
        stage ('ecs update'){
            steps {
                script {
                    withAWS(region:'us-east-2') {
                        def ecr_image="020046395185.dkr.ecr.us-east-2.amazonaws.com/tweet:${GIT_COMMIT}"
                        def task_definition= sh "aws ecs describe-task-definition --task-definition "${params.family}" --region "us-east-2""
                        def new_task_definition=sh "echo ${task_definition} | jq --arg IMAGE "${ecr_image}" '.taskDefinition | .containerDefinitions[0].image = ${IMAGE} | del(.taskDefinitionArn) | del(.revision) | del(.status) | del(.requiresAttributes) | del(.compatibilities)'"
                        def new_task_info=sh "aws ecs register-task-definition --region "us-east-2" --cli-input-json "${new_task_definition}""
                        def new_revision=sh "echo ${new_task_info} | jq '.taskDefinition.revision'"
                        sh "aws ecs update-service --cluster ${params.ecs_cluster} --service ${params.service_name} --task-definition ${params.family}:${new_revision}"
                    }
                }
                
            }
        }

    }
}




























