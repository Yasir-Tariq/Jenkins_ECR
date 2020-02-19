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
            sh "aws ecr get-login --no-include-email --region us-east-2"
            sh "docker build -t tweet ."
            sh "docker tag tweet:latest 020046395185.dkr.ecr.us-east-2.amazonaws.com/tweet:${GIT_COMMIT}"
            sh "docker push 020046395185.dkr.ecr.us-east-2.amazonaws.com/tweet:${GIT_COMMIT}"
              }
        }
        stage ('ecs update'){
            steps {
                ecr_image="020046395185.dkr.ecr.us-east-2.amazonaws.com/tweet:${GIT_COMMIT}"
                task_definition=$(aws ecs describe-task-definition --task-definition "${params.family}" --region "us-east-2")
                new_task_definition=$(echo $task_definition | jq --arg IMAGE "${ecr_image}" '.taskDefinition | .containerDefinitions[0].image = ${IMAGE} | del(.taskDefinitionArn) | del(.revision) | del(.status) | del(.requiresAttributes) | del(.compatibilities)')
                new_task_info=$(aws ecs register-task-definition --region "us-east-2" --cli-input-json "${new_task_definition}")
                new_revision=$(echo ${new_task_info} | jq '.taskDefinition.revision')
                sh "aws ecs update-service --cluster ${params.ecs_cluster} --service ${params.service_name} --task-definition ${params.family}:${new_revision}"
            }
        }

    }
}





