# aws-ecs-fargate
Tutorial: AWS ECS Fargate via CLI

## Tutorial: Creating a Cluster with a Fargate Task Using the Amazon ECS CLI
##### REF: https://docs.aws.amazon.com/AmazonECS/latest/developerguide/ecs-cli-tutorial-fargate.html

## Prerequisites

##### A.) Functional AWS CLI (likely an administrative account with API access)
##### REF: https://aws.amazon.com/cli/
##### Note: Creating AWS credentials - https://serverless.com/framework/docs/providers/aws/guide/credentials/
```
pip install awscli
aws --version
aws configure
aws iam get-user
```

##### B.) Functional Amazon ECS CLI
##### REF: https://docs.aws.amazon.com/AmazonECS/latest/developerguide/ECS_CLI_installation.html
##### Note: Creating AWS credentials - https://serverless.com/framework/docs/providers/aws/guide/credentials/
```
# For macOS:
sudo curl -o /usr/local/bin/ecs-cli https://s3.amazonaws.com/amazon-ecs-cli/ecs-cli-darwin-amd64-latest
sudo chmod +x /usr/local/bin/ecs-cli

# For Linux systems:
sudo curl -o /usr/local/bin/ecs-cli https://s3.amazonaws.com/amazon-ecs-cli/ecs-cli-linux-amd64-latest
sudo chmod +x /usr/local/bin/ecs-cli

# For Windows systems using Windows PowerShell (as an administrator):
New-Item ‘C:\Program Files\Amazon\ECSCLI’ -type directory
Invoke-WebRequest -OutFile ‘C:\Program Files\Amazon\ECSCLI\ecs-cli.exe’ https://s3.amazonaws.com/amazon-ecs-cli/ecs-cli-windows-amd64-latest.exe
# Note: Add C:\Program Files\Amazon\ECSCLI to the Windows PATH and then restart PowerShell


# Verify the CLI version (all OSes):
ecs-cli --version
```
##### OPTIONAL: Configuring the Amazon ECS CLI
##### REF: https://docs.aws.amazon.com/AmazonECS/latest/developerguide/ECS_CLI_Configuration.html


## Deploy

##### 1.) Edit the configuration for your specific deployment secrets and upload to AWS
```
################################################################################
# Create the task execution role:
aws iam --region us-east-1 create-role --role-name ecsTaskExecutionRole --assume-role-policy-document file://task-execution-assume-role.json

# Attach the task execution role policy:
aws iam --region us-east-1 attach-role-policy --role-name ecsTaskExecutionRole --policy-arn arn:aws:iam::aws:policy/service-role/AmazonECSTaskExecutionRolePolicy

# Create a cluster configuration:
ecs-cli configure --cluster tutorial --region us-east-1 --default-launch-type FARGATE --config-name tutorial

# Create a CLI profile using your access key and secret key:
# Note: Replace a and a with your specific/desired values!
ecs-cli configure profile --access-key AWS_ACCESS_KEY_ID --secret-key AWS_SECRET_ACCESS_KEY --profile-name tutorial

# Create an Amazon ECS cluster with the ecs-cli up command:
ecs-cli up
# NOTE: Record these three (3) output value IDs for upcoming/future commands: VPC and Subnet(s)
# VPC created: vpc-XXXXXXXXXXXXXXXXX
# Subnet created: subnet-XXXXXXXXXXXXXXXXX
# Subnet created: subnet-XXXXXXXXXXXXXXXXX

# Create a security group using the VPC ID from previous output above:
aws ec2 create-security-group --group-name "my-sg" --description "My security group" --vpc-id "VPC_ID"
# NOTE: Record the GroupID output for upcoming/future commands
# sg-XXXXXXXXXXXXXXXXX

# Add a security group rule using the GroupID from previous output above to allow inbound access on port 80:
aws ec2 authorize-security-group-ingress --group-id "security_group_id" --protocol tcp --port 80 --cidr 0.0.0.0/0

# Edit ecs-params.yml
# Change/update these three (3) values using the results from output previously recorded above:
# "subnet ID 1" and "subnet ID 2" and "security group ID"
vi ecs-params.yml


################################################################################
# Deploy your cluster with ecs-cli compose service up:
ecs-cli compose --project-name tutorial service up --create-log-groups --cluster-config tutorial

# View the containers that are running in the service with ecs-cli compose service ps:
ecs-cli compose --project-name tutorial service ps --cluster-config tutorial
# Note1: Point your web browser at that address displayed in the output (you should see the WordPress installation wizard)
# Note2: Record the task-id value displayed in the output for the container for upcoming/future commands
#        Example task-id value might look like: a06a6642-12c5-4006-b1d1-033994580605

# View the logs for the task
ecs-cli logs --task-id TaskIdFromPreviousOutputGoesHere --follow --cluster-config tutorial
# CTRL+C to quit


################################################################################
# OPTIONAL: Scale the Tasks on the Cluster:
ecs-cli compose --project-name tutorial service scale 2 --cluster-config tutorial

# Now you should see two more containers in your cluster:
ecs-cli compose --project-name tutorial service ps --cluster-config tutorial


################################################################################
# Clean up your resources so they do not incur any more charges!

# FIRST: Delete the service so that it stops the existing containers and does not try to run any more tasks:
ecs-cli compose --project-name tutorial service down --cluster-config tutorial

# SECOND: Ttake down your cluster (which cleans up the resources that you created earlier with ecs-cli up)
ecs-cli down --force --cluster-config tutorial

```