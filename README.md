# cloudformation-aws-asg-demo

A simple CloudFormation Stack to demonstrate deployment of an EC2 autoscaling group accessible via SSM in an existing (default) VPC using the aws cli. In a production environment we would expect to use Codecommit, CodeBuild, CodePipeline etc. Presumes we are using a 'nix workstation.

- [cloudformation-aws-asg-demo](#cloudformation-aws-asg-demo)
  - [Populate environment variables](#populate-environment-variables)
  - [Validate template](#validate-template)
  - [create a Change Set using the AWS CLI:](#create-a-change-set-using-the-aws-cli)
    - [For a new stack:](#for-a-new-stack)
    - [For an existing (deployed) stack:](#for-an-existing-deployed-stack)
  - [Create (deploy) a new stack:](#create-deploy-a-new-stack)
  - [Updating an Existing Stack:](#updating-an-existing-stack)
  - [Deleting the Stack](#deleting-the-stack)

## Populate environment variables

```bash
. ./setenv.sh 
```

## Validate template

```bash
aws cloudformation validate-template --template-body file://autoscaling.yaml
```

## create a Change Set using the AWS CLI:

We do this to review the impact of our changes before deploying them

### For a new stack:

Interpolating environment variables to aws cli commands:

```bash
aws cloudformation create-change-set \
    --stack-name AutoScalingGroupStackThis \
    --template-body file://autoscaling.yaml \
    --change-set-name MyChangeSet \
    --parameters ParameterKey=InstanceType,ParameterValue=$INSTANCE_TYPE \
                 ParameterKey=VPCSubnetIds,ParameterValue=\"$VPC_SUBNET_IDS\" \
                 ParameterKey=AllowedCIDR,ParameterValue=$ALLOWED_CIDR \
                 ParameterKey=VPCId,ParameterValue=$VPC_ID \
    --tags Key=Environment,Value=test Key=Project,Value=CloudFormationDemo \
    --capabilities CAPABILITY_NAMED_IAM \
    --change-set-type CREATE
```

### For an existing (deployed) stack:

```bash
aws cloudformation create-change-set \
    --stack-name AutoScalingGroupStackThis \
    --template-body file://autoscaling.yaml \
    --change-set-name MyChangeSet \
     --parameters ParameterKey=InstanceType,ParameterValue=$INSTANCE_TYPE \
                 ParameterKey=VPCSubnetIds,ParameterValue=\"$VPC_SUBNET_IDS\" \
                 ParameterKey=AllowedCIDR,ParameterValue=$ALLOWED_CIDR \
                 ParameterKey=VPCId,ParameterValue=$VPC_ID \
    --tags Key=Environment,Value=test Key=Project,Value=CloudFormationDemo \
    --capabilities CAPABILITY_NAMED_IAM
```

After creating the Change Set, we can review the changes either in the console or via the CLI, e.g. with:

```bash
aws cloudformation describe-change-set --change-set-name MyChangeSet --stack-name AutoScalingGroupStackThis
```

If we've recently created a Change Set for this stack and it hasn't been executed yet, the stack will be in a "REVIEW_IN_PROGRESS" state. We must either execute or delete the Change Set before we can update the stack.
To execute the Change Set:

```bash
aws cloudformation execute-change-set --change-set-name MyChangeSet --stack-name AutoScalingGroupStackThis
```

If we decide not to proceed with the Change Set, we can delete it:

```bash
aws cloudformation delete-change-set --change-set-name MyChangeSet --stack-name AutoScalingGroupStackThis
```

## Create (deploy) a new stack:

```bash
aws cloudformation create-stack \
    --stack-name AutoScalingGroupStackThis \
    --template-body file://autoscaling.yaml \
    --parameters ParameterKey=InstanceType,ParameterValue=$INSTANCE_TYPE \
                 ParameterKey=VPCSubnetIds,ParameterValue=\"$VPC_SUBNET_IDS\" \
                 ParameterKey=AllowedCIDR,ParameterValue=$ALLOWED_CIDR \
                 ParameterKey=VPCId,ParameterValue=$VPC_ID \
    --capabilities CAPABILITY_NAMED_IAM \
    --region $AWS_REGION \
    --tags Key=Environment,Value=test Key=Project,Value=CloudFormationDemo
```

## Updating an Existing Stack:

```bash
aws cloudformation update-stack \
    --stack-name AutoScalingGroupStackThis \
    --template-body file://autoscaling.yaml \
    --parameters ParameterKey=InstanceType,ParameterValue=$INSTANCE_TYPE \
                 ParameterKey=VPCSubnetIds,ParameterValue=\"$VPC_SUBNET_IDS\" \
                 ParameterKey=AllowedCIDR,ParameterValue=$ALLOWED_CIDR \
                 ParameterKey=VPCId,ParameterValue=$VPC_ID \
    --capabilities CAPABILITY_NAMED_IAM \
    --region $AWS_REGION \
    --tags Key=Environment,Value=test Key=Project,Value=CloudFormationDemo
```

## Deleting the Stack

To clean up:

```bash
aws cloudformation delete-stack --stack-name AutoScalingGroupStackThis
```
