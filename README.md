# Micro service architecture with AWS ECS and AutoScaling

## Prerequisite

- git
- docker
- aws-cli

### Create ECR repository and push image to it

```shell
$ aws cloudformation create-stack --stack-name sampleECRRepository --template-body file://ecrRepository.yaml --no-disable-rollback --parameters ParameterKey=RepositoryName,ParameterValue=ecr_repository_name
$ aws ecr get-login --no-include-email --region ${AWS.Region}
$ docker login -u AWS -p XXXXXX.......
$ git clone https://github.com/aws-samples/ecs-demo-php-simple-app.git
$ cd ecs-demo-php-simple-app
$ docker build -t ecr_repository_name .
$ docker tag ecr_repository_name:latest ${AWS.AccountId}.dkr.ecr.${AWS.Region}.amazonaws.com/ecr_repository_name:latest
$ docker push ${AWS.AccountId}.dkr.ecr.${AWS.Region}.amazonaws.com/ecr_repository_name:latest
```
Then the image will be pushed to the ECR repository.

### Create ECS Cluster, TaskDefinition, and Service with AutoScaling EC2 containers

`ECRRepositoryArn`, the output of sampleECRRepository, is needed to be set ContainerDefinitions.ContainerDefinitions.Image in microservices.yaml like below.
```yaml
ContainerDefinitions:
  Type: AWS::ECS::TaskDefinition
  Properties:
    ContainerDefinitions:
      - Image: '${ECRRepositoryArn}:latest'
```
Finally, create AutoScalingGroup, ECS Cluster, TaskDefinition, and Service.
```shell
$ aws cloudformation create-stack --stack-name sampleECSClusterWithAutoScaling --template-body file://microservices.yaml --no-disable-rollback --capabilities CAPABILITY_NAMED_IAM
```
