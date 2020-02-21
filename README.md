# Jenkins_Automated_ECR_ECS
Updating AWS ECR repository with update image and attaching it to the ECS service's tasks.


## Pre Reqs

Following pre requisites are necessary for proper understanding:
- AWS services knowledge
- Groovy syntax for Jenkins Pipelines
- Docker knowledge
- Jenkins knowledge
- jq tool
- Shell commands (aws ecs cli)

## Plugins
Following plugin is used in this project:
- Pipeline:AWS Steps



## Files
This repository consists of following files:

1.Jenkinsfile: contains Jenkins Pipeline code.
2.Dockerfile: contains a sample web application image.
3.index.html: contains code for the application.
4.linux.png: a sample picture to be attached to the web application.
Please note that the ECS infrastructure ECR repository would be provisioned before hand.


## Explanation

In this project, we are using the "Declarative" approach for building the Jenkins pipeline.
Therefore, Source Code Management (SCM) is used to get this repository to be fed to the
Jenkins Pipeline. Also, we will be using Git Webhooks to notify Jenkins about the latest push
event so that the build automatically triggers itself whenever any git change occurs.
You can add a webhook in the repository settings and the URL would be the Jenkins URL followed
by the port 8080.
e.g, http://some-url:8080/github-webhook/

The scenario of this project is that whenever there is some change in the Dockerfile or the 
index.html file, a job will be build for building the image and pusing it to AWS ECR repository
with the latest Git commit hash value. There is a stage for this process. Next there is a stage
for updating the AWS ECS resources. For this, we are using a bunch of shell commands and jq tool
which will do the job for us using aws ecs-cli. Below is the scripting:

## Script
    sh '''ECR_IMAGE="020046395185.dkr.ecr.\${rgn}.amazonaws.com/tweet:\${GIT_COMMIT}"
                        TASK_DEFINITION=$(aws ecs describe-task-definition \
                        --task-definition \${fam} --region \${rgn})\
                        NEW_TASK_DEFINTIION=$(echo \$TASK_DEFINITION | jq \
                        --arg IMAGE "\$ECR_IMAGE" \'.taskDefinition | \
                        .containerDefinitions[0].image = \$IMAGE |\
                        del(.taskDefinitionArn) | del(.revision) |\
                        del(.status) | del(.requiresAttributes) | \
                        del(.compatibilities)\')
                        NEW_TASK_INFO=$(aws ecs register-task-definition
                        --region \${rgn} --cli-input-json "\$NEW_TASK_DEFINTIION")
                        NEW_REVISION=$(echo \$NEW_TASK_INFO | jq \'.taskDefinition.revision\')
                        aws ecs update-service --cluster \${cluster} --service \${svc} \
                        --task-definition \${fam}:\${NEW_REVISION} 
'''

We will get the current task definition, and update the output of it according to the latest
git-tagged image from the AWS ECR and then we will update the AWS ECS service. This will create
a new revision for the task definition and attach it to our service accordingly.



