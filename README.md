# Micro service architecture with AWS ECS and AutoScaling

## Prerequisite

- git
- docker
- aws-cli

## Create ECR repository and push image to it

```shell
$ aws cloudformation create-stack --stack-name sampleECRRepository --template-body file://ecrRepository.yaml --no-disable-rollback --parameters ParameterKey=RepositoryName,ParameterValue=ecr_repository_name
$ aws ecr get-login --no-include-email --region ${AWS.Region}
$ docker login -u AWS -p XXXXXX.......
$ git clone https://github.com/aws-samples/ecs-demo-php-simple-app.git
$ cd ecs-demo-php-simple-app
```

Before `docker build`, just put `healthcheck.php` in `ecs-demo-php-simple-app/src` like `ecs-demo-php-simple-app/src/healthcheck.php` for ECS to check the health of container instances.

ECS will see if container instances are regularly running with the helth-check accessing `/healthcheck.php`.

```shell
$ docker build -t ecr_repository_name .
$ docker tag ecr_repository_name:latest ${AWS.AccountId}.dkr.ecr.${AWS.Region}.amazonaws.com/ecr_repository_name:latest
$ docker push ${AWS.AccountId}.dkr.ecr.${AWS.Region}.amazonaws.com/ecr_repository_name:latest
```
Then the image will be pushed to the ECR repository.

## Create ECS Cluster, TaskDefinition, and Service with AutoScaling EC2 containers

`ECRRepositoryArn`, the output of sampleECRRepository, is needed to be set as ContainerDefinitions.ContainerDefinitions.Image in microservices.yaml like below.
```yaml
ECSTaskDefinition:
  Type: AWS::ECS::TaskDefinition
  Properties:
    ContainerDefinitions:
      - Image: '${ECRRepositoryArn}:latest'
```
Finally, create AutoScalingGroup, ECS Cluster, TaskDefinition, and Service.
```shell
$ aws cloudformation create-stack --stack-name sampleECSClusterWithAutoScaling --template-body file://microservices.yaml --no-disable-rollback --capabilities CAPABILITY_NAMED_IAM
```

Now, the container instances and tasks will be running even if these instances and tasks are stopped on either accident or purpose.
