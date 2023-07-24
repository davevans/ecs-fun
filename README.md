#Commands

Show all profiles configured
`aws configure list-profiles`

Show profile working
`aws sts get-caller-identity --profile DavCICD`

Create a policy with ECS permissions
`aws iam create-policy --policy-name ecs-policy --description "Dav's ECS policy" --policy-document file://ECS.json`
`aws iam create-policy-version --policy-arn arn:aws:iam::289410895629:policy/ecs-policy --policy-document file://ECS.json --set-as-default` ## Update the policy

Create role that can deploy cloudformation/iam/ecs
`aws cloudformation create-stack/update-stack --stack-name "dav-cicd-stack" --template-body file://CICD.yaml --capabilities CAPABILITY_NAMED_IAM`


Note that when using `--profile DavCICD` the profile in .aws config file is configured to use the `arn:aws:iam::289410895629:role/deployer-role` role created below

Create an ECS cluster using arn:aws:iam::289410895629:role/DavCICDRole role
`aws cloudformation create-stack/update-stack --profile DavCICD --stack-name "dav-ecs-stack" --template-body file://EcsStack.yaml --capabilities CAPABILITY_NAMED_IAM`

Create task definition
`aws ecs register-task-definition --profile DavCICD --cli-input-json file://colour-app-task-definition.json`

Create service
`aws ecs create-service --profile DavCICD --cluster dav-ecs-cluster --service-name colour-app-service --network-configuration "awsvpcConfiguration={subnets=[subnet-87be29df],securityGroups=[sg-0241476dbc1013033, sg-056ff6840df673e2f],assignPublicIp=ENABLED}" --task-definition colour-app --launch-type FARGATE --desired-count 1 --scheduling-strategy REPLICA`

`aws ecs create-service --profile DavCICD --cli-input-json file://colour-app-service.json`

Update Service
`aws ecs update-service --profile DavCICD --cluster dav-ecs-cluster --service colour-app-service --task-definition colour-app:3`

#Update to a new version of the app
## Create new task definition with image set to new image version
`aws ecs register-task-definition --profile DavCICD --cli-input-json file://colour-app-task-definition.json`

## Update service (will trigger rolling deployment), specifying new revision of task definition
aws ecs update-service --profile DavCICD --cluster dav-ecs-cluster --service colour-app-service --task-definition colour-app:3


#Clean up

aws ecs delete-service --cluster dav-ecs-cluster --service colour-app-service --force
aws ecs delete-cluster --cluster dav-ecs-cluster colour-app-service 