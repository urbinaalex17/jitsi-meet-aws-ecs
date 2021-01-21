
<p align="center">
  <img src="https://img.s3wfg.com/web/img/images_uploaded/8/c/dominion_logo_620x350.jpg" width="30%" height="30%"/>
</p>

# Jitsi Meet on Amazon Elastic Container Service

<img src="docs/jitsi-docker.png" width="25%" height="25%"/>

[Jitsi](https://jitsi.org/) is a set of Open Source projects that allows you to easily build and deploy secure videoconferencing solutions.

[Jitsi Meet](https://jitsi.org/jitsi-meet/) is a fully encrypted, 100% Open Source video conferencing solution that you can use all day, every day, for free with no account needed.

This repository contains the necessary tools to run a Jitsi Meet stack on [Docker](https://www.docker.com) using [Docker Compose](https://docs.docker.com/compose/), as well as the instructions needed to deployment the stack in [Amazon Elastic Container Services](https://aws.amazon.com/ecs/)

The following high-level design architecture diagran depicts the services, containers and ports exposed by Jitsi on ECS:

<img src="docs/aws-ecs-jitsi-architecture.png" width="90%" height="90%"/>

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

[Generate an access key](https://docs.aws.amazon.com/IAM/latest/UserGuide/id_credentials_access-keys.html#Using_CreateAccessKey) in the AWS Console and [set up](https://docs.aws.amazon.com/AmazonECS/latest/developerguide/ECS_CLI_Configuration.html) the ecs-cli to use it.

The following environment variables will be set in the shell:

| Name | Description | Example |
| --- | --- | --- |
| PROFILE_NAME | Specifies the profile name to use | aws |
| AWS_ACCESS_KEY_ID | Specifies the AWS access key to use | AKIAXKO5CYVQ3XXXXXXX |
| AWS_SECRET_ACCESS_KEY | Specifies the AWS secret key to use | KHvshFRp8SstYaou936ZtQD0IaaZXXXXXXXXXXXX |
| ECS_CLUSTER | Specifies the ECS cluster name to use | jitsi-cluster |
| AWS_REGION | Specifies the AWS region to use | eu-west-3 |
| AWS_KEYPAIR | Specifies the name of an existing key pair to enable SSH access to the EC2 instances | jitsi-keypair | 
| AWS_INSTANCE_TYPE | Specifies the EC2 instance type for your container instances | m5.xlarge |
| ECS_CLUSTER_SIZE | Specifies the number of instances to launch and register to the cluster | 2 |


Configure ECS Credentials
```bash
export PROFILE_NAME=<replace>
export AWS_ACCESS_KEY_ID=<replace>
export AWS_SECRET_ACCESS_KEY=<replace>
ecs-cli configure profile --profile-name $PROFILE_NAME --access-key $AWS_ACCESS_KEY_ID --secret-key $AWS_SECRET_ACCESS_KEY
```
 
Configure ECS cluster
```bash
export ECS_CLUSTER=<replace>
export AWS_REGION=<replace>
ecs-cli configure --cluster $ECS_CLUSTER --default-launch-type EC2 --region $AWS_REGION --config-name $ECS_CLUSTER
```

Create ECS cluster
```bash
export AWS_KEYPAIR=<replace>
export AWS_INSTANCE_TYPE=<replace>
export ECS_CLUSTER_SIZE=<replace>
ecs-cli up --keypair $AWS_KEYPAIR --capability-iam --size $ECS_CLUSTER_SIZE --instance-type $AWS_INSTANCE_TYPE --launch-type EC2
```

### ECS Task Definitions

After getting the ECS cluster up and running, the task definition for jitsi meet must be created using the [docker-compose.yml](jitsi-meet/docker-compose.yml) file located in [jitsi-meet](jitsi-meet) directory.

**WARNING: If the task definition is already created and/or manually modified through the AWS Console, executing the following command will overwrite the task definition, leading to lost any previous configuration.**

```bash
cd jitsi-meet
cp env.example .env
./gen-passwords.sh
ecs-cli compose --file docker-compose.yml create
```

Through the AWS Console, go to Elastic Container Services, select the task definition *jitsi-meet*, select the latest revision and create a new one. Edit the container definition of *jicofo, jvb y web* in the Network Settings section and fill out the *Links* parameter with the following content:

**prosody:xmpp.meet.jitsi**

Click on *Create* to save the new task definition.

<img src="docs/aws-network-settings-links-1.png" width="75%" height="75%"/>

Repeat the last procedure, now for *prosody* container definition but in the *Hostname* parameter with the following content:

**xmpp.meet.jitsi**

Click on *Create* to save the new task definition.

<img src="docs/aws-network-settings-links-2.png" width="75%" height="75%"/>

### Create ECS Service

Through the AWS Console, go to Elastic Container Service, select the ECS Cluster, in the *Services* tab, click on create a new service and configure it as follows:

- In Lauch Type select *EC2*
- Fill out Service name with the content *jitsi-meet*
- Fill out Number of tasks with the content *2* (It should be the same number as defined in ECS_CLUSTER_SIZE)
- In Task Placement select *One Task per host*

Click on Next Step three times and finally click on *Create Service*

<img src="docs/aws-create-ecs-service.gif" width="75%" height="75%"/>

### Update default ECS Security Group

The following ports must be open in the default security group:

| Type | Protocol | Port Range |
| --- | --- | --- |
| HTTP | TCP | 80 |
| Custom TCP | TCP | 8080 | 
| Custom TCP | TCP | 8000 |
| Custom TCP | TCP | 8443 |
| Custom UDP | UDP | 10000 |

Select the ECS cluster, in the ECS Instance tab, click on any ECS Instance, select the EC2 instance again, in the security tab, click on the security group to edit the inbound rules.

<img src="docs/aws-update-default-ecs-sg.gif" width="75%" height="75%"/>

### Configure an Application Load Balancer with SSL

- Listeners: Add HTTP (Secure HTTP) listener 
- VPC: Select the VPC created for the ECS Cluster and select both availability zones.
- Upload the certificated or select it from ACM.
- Security group: Add the port 443 and 80-
- Target:
    - Protocol: HTTPS
    - Port: 8443
        - Health check: 
            - Protocol: HTTPS
            - Path: /
- Register Target: Select just one ECS instances from the ECS cluster.
