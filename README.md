# prometheus_aws_ecs

Description of running the prom/prometheus docker container in AWS ECS as a Fargate launch type, i.e. runs serverless/without EC2.  Went down this path to hang an instance on WWW to get a better understanding of prometheus.io (coming from a Nagios background), but mostly to begin collecting data for data analytics.

## prereqs
- ensure you have an AWS account 
- ensure docker is installed on your build box; the mac way https://docs.docker.com/docker-for-mac/install/
- ensure the aws cli is installed on your build box

## obtain docker container
wanted to focus on ECS vs building containers, so just pulled from https://hub.docker.com/r/prom/prometheus/ using:

`docker pull prom/prometheus`

you can list your docker images using:

`docker images`

output:
> REPOSITORY                                                TAG                 IMAGE ID            CREATED             SIZE
  prom/prometheus                                           latest              67141fa03496        4 weeks ago         80.2MB

should you want to run the container locally:

`docker run --name prometheus -d -p 127.0.0.1:9090:9090 prom/prometheus`

view your local container:

`docker ps`
> http://127.0.0.1:9090/graph

## create an ECS repo
- was my first time, so used the console
- called the repo **prometheus**
- push commands are provided...go ahead and run the `aws ecr get-login ...` command

## tag and upload container to ECS repo
`docker tag prom/prometheus:latest <AWS Account Number>.dkr.ecr.us-east-1.amazonaws.com/prometheus:latest`

`docker push <AWS Account Number>.dkr.ecr.us-east-1.amazonaws.com/prometheus`

check out your images:
`docker images`
> REPOSITORY                                                		TAG                 IMAGE ID            CREATED             SIZE
> <AWS Account Number>.dkr.ecr.us-east-1.amazonaws.com/prometheus   	latest              67141fa03496        4 weeks ago         80.2MB ---- ECS
> prom/prometheus                                           		latest              67141fa03496        4 weeks ago         80.2MB ---- local

## ECS - create task definition
- again, first time, so used console; now have a JSON to build another w/ via cli or CloudFormation
- settings of merit:
-- created in VPC/subnet w/ IG/Public IP capabilities
-- FARGATE
-- ecsTaskExecutionRole IAM Role was created by the task def builder
-- container -- associate with <AWS Account Number>.dkr.ecr.us-east-1.amazonaws.com/prometheus:latest 

## ECS - create cluster
- again, first time, so used console
- settings of merit:
-- FARGATE
-- Services -- create and map to task definition
--- creates AWSServiceRoleForECS IAM Role

## Viewing status of container build
- CloudWatch Logs > LogGroup > /ecs/prometheus

# Interacting with the ECS container

## http
- go find the ENI in VPC associated with ecs, then `http://<Public IP>:9090/graph`

## docker cli
`docker exec <IMAGE ID> <CMD>`
e.g. list the contents of /etc/prometheus/prometheus.yml with `docker exec <IMAGE ID> cat /etc/prometheus/prometheus.yml`

