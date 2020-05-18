---
title: "Fargate Basics"
date: 2020-05-18T14:06:02+02:00
draft: true
---


List of steps

1. Build or create local image of app

2. Create repository on ECR
aws ecr create-repository --repository-name repo-name

//needed for for login to repository
aws ecr get-login --no-include-email | sh

aws ecr describe-repositories --repository-name repo-name

docker tag repo-name url-repo:tag

docker push

3. Define VPC

4. ECS cluster, cloudwatch, security groups

5. Stack for containers task
  
