
<p align="center">
  <img src="https://img.s3wfg.com/web/img/images_uploaded/8/c/dominion_logo_620x350.jpg" />
</p>

# Jitsi Meet on Amazon Elastic Container Service

![](docs/jitsi-docker.png)

[Jitsi](https://jitsi.org/) is a set of Open Source projects that allows you to easily build and deploy secure videoconferencing solutions.

[Jitsi Meet](https://jitsi.org/jitsi-meet/) is a fully encrypted, 100% Open Source video conferencing solution that you can use all day, every day, for free — with no account needed.

This repository contains the necessary tools to run a Jitsi Meet stack on [Docker](https://www.docker.com) using [Docker Compose](https://docs.docker.com/compose/).

## Getting Started

### Prerequisites

You have got to install the following tools: 
- [ecs-cli](https://docs.aws.amazon.com/AmazonECS/latest/developerguide/ECS_CLI_installation.html)
- [docker](https://docs.docker.com/get-docker)
- [docker-compose](https://docs.docker.com/compose/install)

Whether you use a IAM service account or IAM user it has to have the following permissions/roles:
- AmazonECS_FullAccess
- ElasticLoadBalancingFullAccess

### Setting up Elastic Container Service

#### ECS Cluster
Generate a Key in the AWS Console and [set up](https://docs.aws.amazon.com/AmazonECS/latest/developerguide/ECS_CLI_Configuration.html) the ecs-cli to use it as follows:

Configure ECS Credentials
```bash
export PROFILE_NAME=<replace>
export AWS_ACCESS_KEY_ID=<replace>
export AWS_SECRET_ACCESS_KEY=<replace>
ecs-cli configure profile --profile-name $PROFILE_NAME --access-key $AWS_ACCESS_KEY_ID --secret-key $AWS_SECRET_ACCESS_KEY
```

### 
Configure ECS cluster
```bash
export ECS_CLUSTER=jitsi-ecs
export AWS_REGION=eu-west-3
ecs-cli configure --cluster $ECS_CLUSTER --default-launch-type EC2 --region $AWS_REGION --config-name $ECS_CLUSTER
```

Create ECS cluster
```bash
export AWS_KEYPAIR=jitsi-ecs
export AWS_INSTANCE_TYPE=m5.xlarge
export ECS_CLUSTER_SIZE=1
ecs-cli up --keypair $AWS_KEYPAIR --capability-iam --size $ECS_CLUSTER_SIZE --instance-type $AWS_INSTANCE_TYPE --launch-type EC2
```

#### ECS Task Definitions

```bash
ecs-cli compose --file docker-compose.yml create
```
## TODO
- Container Links
- Create ECS Service
- Update Security Group
- Add SSL Certificate
- Configure Load Balancer
