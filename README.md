# IacDevops-on-AWS-CICD-defined-through-Terraform-using-CodePipeline-CodeCommit-CodeBuild

##  Introduction

As far as code functionality, my piece handles the following requirements:

- Provision Permission Sets with Access keys into the AWS Account that hosts the AWS IAM service;

- Link these Permission Sets to Groups and AWS Accounts;

- CI/CD to deploy the Permission Sets automatically when a new set or Group/Account link is configured and pushed

to the code repository;

- the code translates into infrastructure which will be built by running terraform commands.

Users assume Roles assigned to Groups that they are part of, which, in turn, are assigned to AWS Accounts.

Permission Sets dictate the level of access a User has.

Different teams will have different Permission Sets assigned for different Accounts.


The challenge comes from making this process a repeatable one with as little human interaction as possible,

and the resptective manager approve manually by Email before deploying to next environement ( staging or production).

Let focus on CI/CD components from the diagram above.

![image](https://github.com/felixdagnon/Devops-on-AWS-CICD-defined-through-Terraform-using-CodePipeline-CodeCommit-CodeBuild/assets/91665833/021157b0-7bd9-418f-aaee-891be1196b0a)

 AWS provides solutions for all of my needs:

- AWS CodeCommit acts as a git server, providing repositories to store code and allows interaction with your code through the git cli;

- AWS CodeBuild acts as a Build environment/engine and is used to execute instructions in multiple stages, to build and pack code in the

shape of a usable artifact;

- AWS CodePipeline is the CI/CD framework that links the other two services together (as well as others) through executable stages;

- AWS S3 to keep any artifacts that result out of a successful Build Stage, for later use and posterity.

  
Before delving into details, let’s first take a look at the picture.

![image](https://github.com/felixdagnon/Devops-on-AWS-CICD-defined-through-Terraform-using-CodePipeline-CodeCommit-CodeBuild/assets/91665833/0c28eda5-ec5a-4642-afda-59561d69e894)

We are going to focus on IaC DevOps with AWS CodePipeline to implemente VPC with WebApp and DB tiers, EC2 instances, 

Bastion host and Security groups, Application Load Balancer, and also Auto Scaling with Launch Templates.

All these resources will be implement with Terraform using TF Config files for all the infrastructure supposed to build.

![image](https://github.com/felixdagnon/Devops-on-AWS-CICD-defined-through-Terraform-using-CodePipeline-CodeCommit-CodeBuild/assets/91665833/67009b84-8a6a-405d-8266-235d2cc26e3b)

First  create GitHub repository and then let’s check into our GitHub repository the Terraform manifests related to our AWS use case.

![image](https://github.com/felixdagnon/Devops-on-AWS-CICD-defined-through-Terraform-using-CodePipeline-CodeCommit-CodeBuild/assets/91665833/2a893932-cf52-4479-aedb-a6b7608bb08b)

The next step, we will create AWS CodePipeline. So as part of that, when we are creating the CodePipeline, we will reference the 

source as our GitHub repository we have created.

We  will create a CodeBuild project for dev environment  in deploy stage.

In this overall implementation, we are not going to see any deploy stage with CodeDeploy or any other tools provided by AWS,

because AWS doesn't have any tools related to pipeline to deploy the Terraform configurations.

For that purpose, we need to leverage the AWS CodeBuild tool as our deploy tool.

We are going to use CodeBuild tool as our CodeDeploy tool to deploy our Terraform configurations in AWS,

or provision our infrastructure using Terraform.

We will also create a manual approval stage in the pipeline and a CodeBuild project for our deploy staging environment.

So here we are going to demonstrate for two environments but we can scale these to multiple environments accordingly.

As a developer, or as a Terraform configuration admin, I will check in all my files to the GitHub repository.

When we will make some change in the code and then push into GitHub repository. immediately CodePipeline will trigger,

and it'll complete the source stage and then it will move to the deploy stage.In deploy stage, it is going to create 

a dev environment in AWS with all resources.


We are going to leverage all these single set of configuration files to create multiple environments excluding the Terraform related variables (dev.tfvars, stag.tfvars, terraform.tfvars).

so whenever we create dev environment, all these resources will be created and it is going to be devdemo1.devopsincloud.com.

![image](https://github.com/felixdagnon/Devops-on-AWS-CICD-defined-through-Terraform-using-CodePipeline-CodeCommit-CodeBuild/assets/91665833/ca67edc2-ca2d-46e1-b3ae-2d3290577af8)


We will see from internet that it will create a DNS record, It will create Application Load Balancer, 

It will create a certificate manager for creating the SSL certificates.

It'll create the Auto Scaling groups with launch templates.

It will create the NAT gateway for VPC and then outbound communication.

It will create related IAM roles, then it will create the instances,

and then it will also create the Simple Notification Service for Auto Scaling group alerts.

All the these ressouces will be created in the dev deploy stage.


And then, the pipeline, once this is successful, moves to the manual approval stage.

Here the request will send, as an email notification to the respective manager who is provided in that SNS notification.

And that respective manager need to approve this respective email, so that it can move on to the next stage in the pipeline

Once  approved then the staging environment will start getting created. And in staging environment also, 

whatever the resources we have configured in the Terraform configurations, the same resources will be created.

it is going to be stagedemo1.devopsincloud.com and all these resources are going to be get created using the AWS CodePipeline.


![image](https://github.com/felixdagnon/Devops-on-AWS-CICD-defined-through-Terraform-using-CodePipeline-CodeCommit-CodeBuild/assets/91665833/f456132f-6af3-4ea7-b13f-93994382802d)


The advantage of using AWS CodePipeline here is we will use only one version of the entire terraform manifests

template for dev environment and then staging environment.

To do so, we are going to create different stuff like dev.conf will reference the dev related Terraform state files, 

![image](https://github.com/felixdagnon/Devops-on-AWS-CICD-defined-through-Terraform-using-CodePipeline-CodeCommit-CodeBuild/assets/91665833/77b71ad5-58a2-4ed3-8f0e-72eb78fcfe14)

and in the same way stag.conf will reference the staging related terraform.tfstate files.

![image](https://github.com/felixdagnon/Devops-on-AWS-CICD-defined-through-Terraform-using-CodePipeline-CodeCommit-CodeBuild/assets/91665833/075f0dde-de44-47b8-bb7d-3ab27e00e49c)

So terraform.tfstate file access the underlying DynanaDB for the real resources whatever it is created in the cloud. 

Which means all the information related to the resources created in the cloud using Terraform is stored inside this tfstate file.

For multiple environments, we are going to manage each environment state by using dev.conf and then stag.conf,

In addition to that, for dev environment, dev.tfvars related environmental variables will be there. 

![image](https://github.com/felixdagnon/Devops-on-AWS-CICD-defined-through-Terraform-using-CodePipeline-CodeCommit-CodeBuild/assets/91665833/f30f6679-2664-4323-be1b-5bfcc40e9a7e)

and for staging environment, stag.tfvars will be there,

![image](https://github.com/felixdagnon/Devops-on-AWS-CICD-defined-through-Terraform-using-CodePipeline-CodeCommit-CodeBuild/assets/91665833/6a91339a-2724-4c59-9bc0-c3a171f5eb37)

and terraform.tfvars will be generic.

![image](https://github.com/felixdagnon/Devops-on-AWS-CICD-defined-through-Terraform-using-CodePipeline-CodeCommit-CodeBuild/assets/91665833/d012aef3-cff9-4df1-ae52-6a9ccb96594b)

We are going to make many changes to all these TF configs files to support the multiple environments like dev, staging or production.

For that we are going to change the naming convention of all of our resources which will have the local.name appended for them.

so that that resource you can easily identify this belongs to this business division, hyphen, environment name, hyphen, and resource 

name (BusinessDivision-EnvironmentName-ResourceName).

We are also going to create Build specification files.Buildspecdev.yml and buildspecstaging.yml related to dev and staging build 

specification files to implement CodePipeline.

buildpec.dev.yml

![image](https://github.com/felixdagnon/Devops-on-AWS-CICD-defined-through-Terraform-using-CodePipeline-CodeCommit-CodeBuild/assets/91665833/404be862-a27b-46b8-8d36-f840c6f8d9d8)

buildpec.stag.yml

![image](https://github.com/felixdagnon/Devops-on-AWS-CICD-defined-through-Terraform-using-CodePipeline-CodeCommit-CodeBuild/assets/91665833/c1dd9956-d6b5-40b3-8656-4610213d8693)


We will implement all these changes step by step.

## Step-01: terraform-manifests 
- Update `terraform-manifests` by creating `Autoscaling-with-Launch-Templates`
- Create`private-key\terraform-key.pem` with your private key with same name.

## Step-02: c1-versions.tf - Terraform Backends
### Step-02-01 Add backend block as below 
```t
  # Adding Backend as S3 for Remote State Storage
  backend "s3" { }  
```
### Step-02-02: Create file named `dev.conf`
```t
bucket = "terraform-on-aws-for-ec2"
key    = "iacdevops/dev/terraform.tfstate"
region = "us-east-1" 
dynamodb_table = "iacdevops-dev-tfstate" 
```
### Step-02-03: Create file named `stag.conf`
```t
bucket = "terraform-on-aws-for-ec2"
key    = "iacdevops/stag/terraform.tfstate"
region = "us-east-1" 
dynamodb_table = "iacdevops-stag-tfstate" 
```

## Step-02-04: Create S3 Bucket related folders for both environments for Terraform State Storage

Go to Services -> S3 -> terraform-on-aws-for-ec2-demo1

- Create Folder iacdevops

![image](https://github.com/felixdagnon/Devops-on-AWS-CICD-defined-through-Terraform-using-CodePipeline-CodeCommit-CodeBuild/assets/91665833/48b36658-2770-44e7-a5d1-08bb0eaa9ccd)

- Create Folder iacdevops\dev

- Create Folder iacdevops\stag

![image](https://github.com/felixdagnon/Devops-on-AWS-CICD-defined-through-Terraform-using-CodePipeline-CodeCommit-CodeBuild/assets/91665833/dcbaee96-96ee-4fb9-9195-cbfc8fb19cd7)

## Step-02-05: Create DynamoDB Tables for Both Environments for Terraform State Locking

- Create Dynamo DB Table for Dev Environment
  
- Table Name: iacdevops-dev-tfstate

- Partition key (Primary Key): LockID (Type as String)
  
- Table settings: Use default settings (checked)

- Click on Create

![image](https://github.com/felixdagnon/Devops-on-AWS-CICD-defined-through-Terraform-using-CodePipeline-CodeCommit-CodeBuild/assets/91665833/482d4ecb-b7d2-4a3c-87e5-79ab125ac2eb)

  
## Create Dynamo DB Table for Staging Environment

- Table Name: iacdevops-stag-tfstate

- Partition key (Primary Key): LockID (Type as String)

- Table settings: Use default settings (checked)

- Click on Create

![image](https://github.com/felixdagnon/Devops-on-AWS-CICD-defined-through-Terraform-using-CodePipeline-CodeCommit-CodeBuild/assets/91665833/aeae8abd-0d99-4dee-8975-e316bba67430)

## Step-03: Pipeline Build Out - Decisions

We have two options here.

Step-03-01: Option-1: Create separate folders per environment and have same TF Config files (c1 to c13) maintained per environment

More work as we need to manage many environment related configs

- Dev - C1 to C13 - Approximate 30 files

- QA - C1 to C13 - Approximate 30 files

- Stg - C1 to C13 - Approximate 30 files

- Prd - C1 to C13 - Approximate 30 files

- DR - C1 to C13 - Approximate 30 files

- Close to 150 files you need to manage changes.
  
For critical projects which you want to isolate as above, Terraform also recommends this approach but its all case to case basis on the

environment we have built, skill level and organization level standards.

## Step-03-02: Option-2: Create only 1 folder and leverage same C1 to C13 files (approx 30 files) across environments.

Only 30 files to manage across Dev, QA, Staging, Production and DR environments.

- We are going to take this `option-2` and build the pipeline for Dev and Staging environments
  
![image](https://github.com/felixdagnon/terraform-Iacdevops-using-aws-codepipeline/assets/91665833/3eaeb769-bfbb-4c0a-8d25-275e334b033f)

## Step-04: Merge vpc.auto.tfvars and ec2instance.auto.tfvars 
- Merge `vpc.auto.tfvars` and `ec2instance.auto.tfvars` to environment specific `.tfvars` example `dev.tfvars` and `stag.tfvats`
- Also we want to leverage same TF Config files across environments.
- We are going to pass the `.tfvars` file as `-var-file` argument to `terraform apply` command
```t
terraform apply -input=false -var-file=dev.tfvars -auto-approve  
```


### Step-04-01: dev.tfvars
```t
# Environment
environment = "dev"
# VPC Variables
vpc_name = "myvpc"
vpc_cidr_block = "10.0.0.0/16"
vpc_availability_zones = ["us-east-1a", "us-east-1b", "us-east-1c"]
vpc_public_subnets = ["10.0.101.0/24", "10.0.102.0/24", "10.0.103.0/24"]
vpc_private_subnets = ["10.0.1.0/24", "10.0.2.0/24", "10.0.3.0/24"]
vpc_database_subnets= ["10.0.151.0/24", "10.0.152.0/24", "10.0.153.0/24"]
vpc_create_database_subnet_group = true 
vpc_create_database_subnet_route_table = true   
vpc_enable_nat_gateway = true  
vpc_single_nat_gateway = true

# EC2 Instance Variables
instance_type = "t3.micro"
instance_keypair = "terraform-key"
private_instance_count = 2
```
### Step-04-01: stag.tfvars
```t
# Environment
environment = "stag"
# VPC Variables
vpc_name = "myvpc"
vpc_cidr_block = "10.0.0.0/16"
vpc_availability_zones = ["us-east-1a", "us-east-1b", "us-east-1c"]
vpc_public_subnets = ["10.0.101.0/24", "10.0.102.0/24", "10.0.103.0/24"]
vpc_private_subnets = ["10.0.1.0/24", "10.0.2.0/24", "10.0.3.0/24"]
vpc_database_subnets= ["10.0.151.0/24", "10.0.152.0/24", "10.0.153.0/24"]
vpc_create_database_subnet_group = true 
vpc_create_database_subnet_route_table = true   
vpc_enable_nat_gateway = true  
vpc_single_nat_gateway = true


# EC2 Instance Variables
instance_type = "t3.micro"
instance_keypair = "terraform-key"
private_instance_count = 2
```
- Remove / Delete the following two files
  - vpc.auto.tfvars
  - ec2instance.auto.tfvars

## Step-05: terraform.tfvars
- `terraform.tfvars` which autoloads for all environment creations will have only generic variables. 
```t
# Generic Variables
aws_region = "us-east-1"
business_divsion = "hr"
```




## Step-06: Provisioner "local-exec"
### Step-06-01: c9-nullresource-provisioners.tf
- Applicable in CodePipeline -> CodeBuild case. 
```t
 provisioner "local-exec" {
    command = "echo VPC created on `date` and VPC ID: ${module.vpc.vpc_id} >> creation-time-vpc-id.txt"
    working_dir = "local-exec-output-files/"
  }
```
### Step-06-02: c8-elasticip.tf
- Applicable in CodePipeline -> CodeBuild case. 
```t
  provisioner "local-exec" {
    command = "echo Destroy time prov `date` >> destroy-time-prov.txt"
    working_dir = "local-exec-output-files/"
    when = destroy
  }  
```

## Step-07: To Support Multiple Environments
### Step-07-01: c5-03-securitygroup-bastionsg.tf
```t
# Append local.name to "public-bastion-sg"  
  name = "${local.name}-public-bastion-sg"
```
### Step-07-02: c5-04-securitygroup-privatesg.tf
```t
# Append local.name to "private-sg"
  name = "${local-name}-private-sg"  
```

### Step-07-03: c5-05-securitygroup-loadbalancersg.tf
```t
# Append local.name to "loadbalancer-sg"
  name = "${local.name}-loadbalancer-sg"  
```

### Step-07-04: Create Variable for DNS Name to support multiple environments
#### Step-07-04-01: c12-route53-dnsregistration.tf
```t
# DNS Name Input Variable
variable "dns_name" {
  description = "DNS Name to support multiple environments"
  type = string   
}
```
#### Step-07-04-02: c12-route53-dnsregistration.tf
```t
# DNS Registration 
resource "aws_route53_record" "apps_dns" {
  zone_id = data.aws_route53_zone.mydomain.zone_id 
  name    = var.dns_name 
  type    = "A"
  alias {
    name                   = module.alb.lb_dns_name
    zone_id                = module.alb.lb_zone_id
    evaluate_target_health = true
  }  
}
```

In my case the domain names change in this step.

I create hosted zone "kalyandemo.com" in Route 53 console

![image](https://github.com/felixdagnon/Devops-on-AWS-CICD-defined-through-Terraform-using-CodePipeline-CodeCommit-CodeBuild/assets/91665833/9e787e84-dfc5-4b2a-b052-90545789eab7)

Let's create dns name record

#### Step-07-04-03: dev.tfvars
```t
# DNS Name
dns_name = "devdemo5.kalyandemo.com"
```

#### Step-07-04-04: stag.tfvars

```t
# DNS Name
dns_name = "stagedemo5.kalyandemo.com"
```
### Step-07-05: c11-acm-certificatemanager.tf

```t
  subject_alternative_names = [
    #"*.kalyandemo.com"
    var.dns_name  
  ]
```

### Step-07-06: c13-02-autoscaling-launchtemplate-resource.tf
```t
# Append local.name to name_prefix
  name_prefix = "${local.name}-"
```
### Step-07-07: c13-02-autoscaling-launchtemplate-resource.tf
```t
# Append Name = local.name
  tag_specifications {
    resource_type = "instance"
    tags = {
      Name = local.name
    }
  }    
```
### Step-07-08: c13-03-autoscaling-resource.tf
```t
# Append local.name to name_prefix
  name_prefix = "${local.name}-"  
```
### Step-07-09: c13-06-autoscaling-ttsp.tf
```t
# Append local.name to name
  name = "${local.name}-avg-cpu-policy-greater-than-xx"
  name = "${local.name}-alb-target-requests-greater-than-yy"  
```

## Step-08: Create Secure Parameters in Parameter Store
### Step-08-01: Create MY_AWS_SECRET_ACCESS_KEY
- Go to Services -> Systems Manager -> Application Management -> Parameter Store -> Create Parameter 
  - Name: /CodeBuild/MY_AWS_ACCESS_KEY_ID
  - Descritpion: My AWS Access Key ID for Terraform CodePipeline Project
  - Tier: Standard
  - Type: Secure String
  - Rest all defaults
  - Value: ABCXXXXDEFXXXXGHXXX

### Step-08-02: Create MY_AWS_SECRET_ACCESS_KEY
- Go to Services -> Systems Manager -> Application Management -> Parameter Store -> Create Parameter 
  - Name: /CodeBuild/MY_AWS_SECRET_ACCESS_KEY
  - Descritpion: My AWS Secret Access Key for Terraform CodePipeline Project
  - Tier: Standard
  - Type: Secure String
  - Rest all defaults
  - Value: abcdefxjkdklsa55dsjlkdjsakj
 
![image](https://github.com/felixdagnon/Devops-on-AWS-CICD-defined-through-Terraform-using-CodePipeline-CodeCommit-CodeBuild/assets/91665833/a85ae67c-48d4-4331-b83a-5a1be23524c9)

## Step-09: buildspec-dev.yml
- Discuss about following Environment variables we are going to pass
- TERRAFORM_VERSION
  - which version of terraform codebuild should use
  - As on today `1.7.3` is latest we will use that
- TF_COMMAND
  - We will use `apply` to create resources
  - We will use `destroy` in CodeBuild Environment 
- AWS_ACCESS_KEY_ID: /CodeBuild/MY_AWS_ACCESS_KEY_ID
  - AWS Access Key ID is safely stored in Parameter Store
- AWS_SECRET_ACCESS_KEY: /CodeBuild/MY_AWS_SECRET_ACCESS_KEY
  - AWS Secret Access Key is safely stored in Parameter Store
```yaml
version: 0.2

env:
  variables:
    TERRAFORM_VERSION: "1.7.3"
    TF_COMMAND: "apply"
    #TF_COMMAND: "destroy"
  parameter-store:
    AWS_ACCESS_KEY_ID: "/CodeBuild/MY_AWS_ACCESS_KEY_ID"
    AWS_SECRET_ACCESS_KEY: "/CodeBuild/MY_AWS_SECRET_ACCESS_KEY"

phases:
  install:
    runtime-versions:
      python: 3.7
    on-failure: ABORT       
    commands:
      - tf_version=$TERRAFORM_VERSION
      - wget https://releases.hashicorp.com/terraform/"$TERRAFORM_VERSION"/terraform_"$TERRAFORM_VERSION"_linux_amd64.zip
      - unzip terraform_"$TERRAFORM_VERSION"_linux_amd64.zip
      - mv terraform /usr/local/bin/
  pre_build:
    on-failure: ABORT     
    commands:
      - echo terraform execution started on `date`            
  build:
    on-failure: ABORT   
    commands:
    # Project-1: AWS VPC, ASG, ALB, Route53, ACM, Security Groups and SNS 
      - cd "$CODEBUILD_SRC_DIR/terraform-manifests"
      - ls -lrt "$CODEBUILD_SRC_DIR/terraform-manifests"
      - terraform --version
      - terraform init -input=false --backend-config=dev.conf
      - terraform validate
      - terraform plan -lock=false -input=false -var-file=dev.tfvars           
      - terraform $TF_COMMAND -input=false -var-file=dev.tfvars -auto-approve  
  post_build:
    on-failure: CONTINUE   
    commands:
      - echo terraform execution completed on `date`         
```

## Step-10: buildspec-stag.yml 
```yaml
version: 0.2

env:
  variables:
    TERRAFORM_VERSION: "1.7.3"
    TF_COMMAND: "apply"
    #TF_COMMAND: "destroy"
  parameter-store:
    AWS_ACCESS_KEY_ID: "/CodeBuild/MY_AWS_ACCESS_KEY_ID"
    AWS_SECRET_ACCESS_KEY: "/CodeBuild/MY_AWS_SECRET_ACCESS_KEY"

phases:
  install:
    runtime-versions:
      python: 3.7
    on-failure: ABORT       
    commands:
      - tf_version=$TERRAFORM_VERSION
      - wget https://releases.hashicorp.com/terraform/"$TERRAFORM_VERSION"/terraform_"$TERRAFORM_VERSION"_linux_amd64.zip
      - unzip terraform_"$TERRAFORM_VERSION"_linux_amd64.zip
      - mv terraform /usr/local/bin/
  pre_build:
    on-failure: ABORT     
    commands:
      - echo terraform execution started on `date`            
  build:
    on-failure: ABORT   
    commands:
    # Project-1: AWS VPC, ASG, ALB, Route53, ACM, Security Groups and SNS 
      - cd "$CODEBUILD_SRC_DIR/terraform-manifests"
      - ls -lrt "$CODEBUILD_SRC_DIR/terraform-manifests"
      - terraform --version
      - terraform init -input=false --backend-config=stag.conf
      - terraform validate
      - terraform plan -lock=false -input=false -var-file=stag.tfvars           
      - terraform $TF_COMMAND -input=false -var-file=stag.tfvars -auto-approve  
  post_build:
    on-failure: CONTINUE   
    commands:
      - echo terraform execution completed on `date`             
```

## Step-11: Create Github Repository and Check-In file
### Step-11-01: Create New Github Repository
- Go to  github.com and login with your credentials 
- Click on **Repositories Tab**
- Click on  **New** to create a new repository 
- **Repository Name:** terraform-iacdevops-with-aws-codepipeline
- **Description:** Implement Terraform IAC DevOps for AWS Project with AWS CodePipeline
- **Repository Type:** Private
- **Choose License:** Apache License 2.0
- Click on **Create Repository**
- Click on **Code** and Copy Repo link

  ![image](https://github.com/felixdagnon/Devops-on-AWS-CICD-defined-through-Terraform-using-CodePipeline-CodeCommit-CodeBuild/assets/91665833/cc2031d5-194e-43da-ae4c-891b9e5a6de4)


I create demo-repos folder in my local

### Step-11-02: Clone Remote Repo and Copy all related files 

![image](https://github.com/felixdagnon/Devops-on-AWS-CICD-defined-through-Terraform-using-CodePipeline-CodeCommit-CodeBuild/assets/91665833/77832046-bfb9-42a3-b9a4-debcedeb18c7)

```t
# Change Directory
cd demo-repos

# Execute Git Clone
git clone https://github.com/felixdagnon/terraform-iacdevops-with-aws-codepipeline.git

# Verify Git Status
git status

# Git Commit
git commit -am "First Commit"

# Push files to Remote Repository
git push

# Verify same on Remote Repository
https://github.com/stacksimplify/terraform-iacdevops-with-aws-codepipeline.git
```

![image](https://github.com/felixdagnon/Devops-on-AWS-CICD-defined-through-Terraform-using-CodePipeline-CodeCommit-CodeBuild/assets/91665833/91532e08-0387-4309-b9a8-f8f5ea73f0a6)


![image](https://github.com/felixdagnon/Devops-on-AWS-CICD-defined-through-Terraform-using-CodePipeline-CodeCommit-CodeBuild/assets/91665833/21dd41ae-d2a2-4af6-b14e-1a52494bc530)


Let's check Github

![image](https://github.com/felixdagnon/Devops-on-AWS-CICD-defined-through-Terraform-using-CodePipeline-CodeCommit-CodeBuild/assets/91665833/28c4c56f-3e5b-4b97-9f32-113439095509)


## Step-12: Verify if AWS Connector for GitHub already installed on your Github

- Go to below url and verify
  
- **URL:** https://github.com/settings/installations

![image](https://github.com/felixdagnon/Devops-on-AWS-CICD-defined-through-Terraform-using-CodePipeline-CodeCommit-CodeBuild/assets/91665833/afd29ee8-d36b-43ee-8331-bc9112f99b54)



## Step-13: Create Github Connection from AWS Developer Tools

- Go to Services -> CodePipeline -> Create Pipeline
  
- In Developer Tools -> Click on **Settings** -> Connections -> Create Connection
  
- **Select Provider:** Github
  
- **Connection Name:** terraform-iacdevops-aws-cp-con1
  
- Click on **Connect to Github**

![image](https://github.com/felixdagnon/Devops-on-AWS-CICD-defined-through-Terraform-using-CodePipeline-CodeCommit-CodeBuild/assets/91665833/8184272d-1260-42d8-92b2-8ebeddd671f8)
  
- GitHub Apps: Click on **Install new app**
  
- It should redirect to github page `Install AWS Connector for GitHub`
  
- **Only select repositories:** terraform-iacdevops-with-aws-codepipeline

![image](https://github.com/felixdagnon/Devops-on-AWS-CICD-defined-through-Terraform-using-CodePipeline-CodeCommit-CodeBuild/assets/91665833/1463cec5-bdfa-46d1-9059-641117250290)


- Click on **save**

  Redirect to AWS console
  
- Click on **Connect**

  ![image](https://github.com/felixdagnon/Devops-on-AWS-CICD-defined-through-Terraform-using-CodePipeline-CodeCommit-CodeBuild/assets/91665833/ba3d4192-4b37-4f43-862b-9e96c95f9efa)

- Verify Connection Status: It should be in **Available** state

![image](https://github.com/felixdagnon/Devops-on-AWS-CICD-defined-through-Terraform-using-CodePipeline-CodeCommit-CodeBuild/assets/91665833/838b3e46-5aec-4032-999f-9dcb3c54ba10)


  
- Go to below url and verify
  
- **URL:** https://github.com/settings/installations

- You should see `Install AWS Connector for GitHub` app installed

![image](https://github.com/felixdagnon/Devops-on-AWS-CICD-defined-through-Terraform-using-CodePipeline-CodeCommit-CodeBuild/assets/91665833/adcb582b-3490-4da0-9809-008e8ae8d41f)


## Step-14: Create AWS CodePipeline

- Go to Services -> CodePipeline -> Create Pipeline
  
### Pipeline settings

- **Pipeline Name:** tf-iacdevops-aws-cp1
  
- **Service role:** New Service Role

- rest all defaults

- Artifact store: Default Location
    
- Encryption Key: Default AWS Managed Key

- Click **Next**
  


![image](https://github.com/felixdagnon/Devops-on-AWS-CICD-defined-through-Terraform-using-CodePipeline-CodeCommit-CodeBuild/assets/91665833/968017b1-a84f-4a8a-a2b3-dff3be74d2b2)


![image](https://github.com/felixdagnon/Devops-on-AWS-CICD-defined-through-Terraform-using-CodePipeline-CodeCommit-CodeBuild/assets/91665833/3ef0af67-8316-4350-9389-5731518db0a2)


### Source Stage

- **Source Provider:** Github (Version 2)
  
- **Connection:** terraform-iacdevops-aws-cp-con1
  
- **Repository name:** terraform-iacdevops-with-aws-codepipeline
  
- **Branch name:** main
  
- **Change detection options:** leave to defaults as checked
  
- **Output artifact format:** leave to defaults as `CodePipeline default`

![image](https://github.com/felixdagnon/Devops-on-AWS-CICD-defined-through-Terraform-using-CodePipeline-CodeCommit-CodeBuild/assets/91665833/06e54c7b-7072-476d-8d86-823050004550)

![image](https://github.com/felixdagnon/Devops-on-AWS-CICD-defined-through-Terraform-using-CodePipeline-CodeCommit-CodeBuild/assets/91665833/c603c3bf-466b-4e50-90ff-60055ee7fcfb)


### Add Build Stage

- **Build Provider:** AWS CodeBuild
  
- **Region:** N.Virginia
  
- **Project Name:** Click on **Create Project**

  ![image](https://github.com/felixdagnon/Devops-on-AWS-CICD-defined-through-Terraform-using-CodePipeline-CodeCommit-CodeBuild/assets/91665833/a9910d1c-9d8d-434c-834e-089cb2bcdd72)

  
  - **Project Name:** codebuild-tf-iacdevops-aws-cp1
    
  - **Description:** CodeBuild Project for Dev Stage of IAC DevOps Terraform Demo

![image](https://github.com/felixdagnon/Devops-on-AWS-CICD-defined-through-Terraform-using-CodePipeline-CodeCommit-CodeBuild/assets/91665833/f5125cfd-75a2-4654-82ca-28d1ec40e23e)


  - **Environment image:** Managed Image
    
  - **Operating System:** Amazon Linux 2
    
  - **Runtimes:** Standard

    ![image](https://github.com/felixdagnon/Devops-on-AWS-CICD-defined-through-Terraform-using-CodePipeline-CodeCommit-CodeBuild/assets/91665833/f4e154b7-76f4-4c9c-9ae3-9ec9e1c258ed)

  - **Image:** latest available today (aws/codebuild/amazonlinux2-x86_64-standard:3.0)
    
  - **Environment Type:** Linux
    
  - **Service Role:** New (leave to defaults including Role Name)

    ![image](https://github.com/felixdagnon/Devops-on-AWS-CICD-defined-through-Terraform-using-CodePipeline-CodeCommit-CodeBuild/assets/91665833/4a78741b-351b-4628-bb28-636799f07c58)

  - **Build specifications:** use a buildspec file
    
  - **Buildspec name - optional:** buildspec-dev.yml  (Ensure that this file is present in root folder of your github repository)

    ![image](https://github.com/felixdagnon/Devops-on-AWS-CICD-defined-through-Terraform-using-CodePipeline-CodeCommit-CodeBuild/assets/91665833/a0cefac3-7b69-49f3-b7e3-58e44ddd943d)

    
  - Rest all leave to defaults
    
  - Click on **Continue to CodePipeline**

![image](https://github.com/felixdagnon/Devops-on-AWS-CICD-defined-through-Terraform-using-CodePipeline-CodeCommit-CodeBuild/assets/91665833/3c04a341-589b-47ce-8494-b501f738e7a9)

    
- **Project Name:** This value should be auto-populated with `codebuild-tf-iacdevops-aws-cp1`
  
- **Build Type:** Single Build


- Click **Next**

  ![image](https://github.com/felixdagnon/Devops-on-AWS-CICD-defined-through-Terraform-using-CodePipeline-CodeCommit-CodeBuild/assets/91665833/0a7af328-61bd-4653-a152-fe086eb7449a)

### Add Deploy Stage

- Click on **Skip Deploy Stage**

 ![image](https://github.com/felixdagnon/Devops-on-AWS-CICD-defined-through-Terraform-using-CodePipeline-CodeCommit-CodeBuild/assets/91665833/9ed6745a-eddf-424c-b9d3-afa02b33d51e)

### Review Stage

- Click on **Create Pipeline**
  
![image](https://github.com/felixdagnon/Devops-on-AWS-CICD-defined-through-Terraform-using-CodePipeline-CodeCommit-CodeBuild/assets/91665833/882482aa-793b-4bc1-bcda-ec65a0e65c8e)

![image](https://github.com/felixdagnon/Devops-on-AWS-CICD-defined-through-Terraform-using-CodePipeline-CodeCommit-CodeBuild/assets/91665833/dac3b543-fcac-4494-9fdf-f581a86237ea)

![image](https://github.com/felixdagnon/Devops-on-AWS-CICD-defined-through-Terraform-using-CodePipeline-CodeCommit-CodeBuild/assets/91665833/e50afffd-f073-46b7-8338-4ab6df779521)


## Step-15: Verify the Pipeline created

- **Verify Source Stage:** Should pass

![image](https://github.com/felixdagnon/Devops-on-AWS-CICD-defined-through-Terraform-using-CodePipeline-CodeCommit-CodeBuild/assets/91665833/b623b5f5-0fac-4b40-8d61-80ae4b2bf9a8)


- **Verify Build Stage:** should fail with error

![image](https://github.com/felixdagnon/Devops-on-AWS-CICD-defined-through-Terraform-using-CodePipeline-CodeCommit-CodeBuild/assets/91665833/cf66d0a6-66c4-4eaf-a4fb-d3078487a41f)
 
  
- Verify Build Stage logs by clicking on **details** in pipeline screen
  
```log
[Container] 2021/05/11 06:24:06 Waiting for agent ping
[Container] 2021/05/11 06:24:09 Waiting for DOWNLOAD_SOURCE
[Container] 2021/05/11 06:24:09 Phase is DOWNLOAD_SOURCE
[Container] 2021/05/11 06:24:09 CODEBUILD_SRC_DIR=/codebuild/output/src851708532/src
[Container] 2021/05/11 06:24:09 YAML location is /codebuild/output/src851708532/src/buildspec-dev.yml
[Container] 2021/05/11 06:24:09 Processing environment variables
[Container] 2021/05/11 06:24:09 Decrypting parameter store environment variables
[Container] 2021/05/11 06:24:09 Phase complete: DOWNLOAD_SOURCE State: FAILED
[Container] 2021/05/11 06:24:09 Phase context status code: Decrypted Variables Error Message: AccessDeniedException: User: arn:aws:sts::180789647333:assumed-role/codebuild-codebuild-tf-iacdevops-aws-cp1-service-role/AWSCodeBuild-97595edc-1db1-4070-97a0-71fa862f0993 is not authorized to perform: ssm:GetParameters on resource: arn:aws:ssm:us-east-1:180789647333:parameter/CodeBuild/MY_AWS_ACCESS_KEY_ID
```


![image](https://github.com/felixdagnon/Devops-on-AWS-CICD-defined-through-Terraform-using-CodePipeline-CodeCommit-CodeBuild/assets/91665833/c67c9033-33d9-44e3-85c0-161538bc712b)

## Step-16: Fix ssm:GetParameters IAM Role issues

### Step-16-01: Get IAM Service Role used by CodeBuild Project

- Get the IAM Service Role name CodeBuild Project is using
  
- Go to CodeBuild -> codebuild-tf-iacdevops-aws-cp1 -> Edit -> Environment
  
- Make a note of Service Role ARN

Here "codebuild-codebuild-tf-iacdevops-aws-cp1-service-role"
   
```t
# CodeBuild Service Role ARN 
arn:aws:iam::180789647333:role/service-role/codebuild-codebuild-tf-iacdevops-aws-cp1-service-role
```

### Step-16-02: Create IAM Policy with Systems Manager Get Parameter Read Permission

- Go to Services -> IAM -> Policies -> Create Policy
  
- **Service:** Systems Manager

- **Actions:** Get Parameters (Under Read)

- **Resources:** All

- Click **Next Tags**

- Click **Next Review**
  
- **Policy name:** systems-manger-get-parameter-access
  
- **Policy Description:** Read Parameters from Parameter Store in AWS Systems Manager Service
  
- Click on **Create Policy**

![image](https://github.com/felixdagnon/Devops-on-AWS-CICD-defined-through-Terraform-using-CodePipeline-CodeCommit-CodeBuild/assets/91665833/294b22f4-5230-462e-8a3d-157eadefd1b7)

### Step-16-03: Associate this Policy to IAM Role

![image](https://github.com/felixdagnon/Devops-on-AWS-CICD-defined-through-Terraform-using-CodePipeline-CodeCommit-CodeBuild/assets/91665833/13d64ded-b0ca-4c82-9133-46d0a25d9ab5)

- Go to Services -> IAM -> Roles -> Search for `codebuild-codebuild-tf-iacdevops-aws-cp1-service-role`
  
- Attach the polic named `systems-manger-get-parameter-access`

![image](https://github.com/felixdagnon/Devops-on-AWS-CICD-defined-through-Terraform-using-CodePipeline-CodeCommit-CodeBuild/assets/91665833/f3f9d304-7a4e-4e1d-bc49-0005ae328eda)


## Step-17: Re-run the CodePipeline 

- Go to Services -> CodePipeline -> tf-iacdevops-aws-cp1
  
- Click on **Release Change**


![image](https://github.com/felixdagnon/Devops-on-AWS-CICD-defined-through-Terraform-using-CodePipeline-CodeCommit-CodeBuild/assets/91665833/e9540763-2399-445f-930d-043494dee424)

  
- **Verify Source Stage:**
  
  - Should pass
    
- **Verify Build Stage:**
  
  - Verify Build Stage logs by clicking on **details** in pipeline screen

 ![image](https://github.com/felixdagnon/Devops-on-AWS-CICD-defined-through-Terraform-using-CodePipeline-CodeCommit-CodeBuild/assets/91665833/84f17894-ab88-4392-9abb-0b1122903fe6)

  - Verify `Cloudwatch -> Log Groups` logs too (Logs saved in CloudWatch for additional reference)

![image](https://github.com/felixdagnon/Devops-on-AWS-CICD-defined-through-Terraform-using-CodePipeline-CodeCommit-CodeBuild/assets/91665833/36b99d1f-cc7d-4c91-80f7-a4b0dbde98a7)

Let's check log event. It shows build and post build state succeeded

![image](https://github.com/felixdagnon/Devops-on-AWS-CICD-defined-through-Terraform-using-CodePipeline-CodeCommit-CodeBuild/assets/91665833/beb57b16-d44d-46c7-bdf3-265525db8e80)

## Step-18: Verify Resources

Let's verify ressoures 

### VPC is created and inside all ressources

![image](https://github.com/felixdagnon/Devops-on-AWS-CICD-defined-through-Terraform-using-CodePipeline-CodeCommit-CodeBuild/assets/91665833/2d355af8-5243-46ce-8a76-c51be9c5a286)

CodeBuild created underline ressourrces for VPC. Let's verify codebuild log

![image](https://github.com/felixdagnon/Devops-on-AWS-CICD-defined-through-Terraform-using-CodePipeline-CodeCommit-CodeBuild/assets/91665833/7979ecfe-7497-4bcf-a1eb-9e12cb9b8e31)

The entire ressources are completed

Download phase complete

![image](https://github.com/felixdagnon/Devops-on-AWS-CICD-defined-through-Terraform-using-CodePipeline-CodeCommit-CodeBuild/assets/91665833/3a6abd0f-3908-4001-8a2c-88c253ec55d4)

Intall and pre build phases completed

![image](https://github.com/felixdagnon/Devops-on-AWS-CICD-defined-through-Terraform-using-CodePipeline-CodeCommit-CodeBuild/assets/91665833/6f2c0f2c-fbea-46f4-a350-f403f6d512c1)

Initializing the backend...

Successfully configured the backend "s3"! Terraform will automatically use this backend.

![image](https://github.com/felixdagnon/Devops-on-AWS-CICD-defined-through-Terraform-using-CodePipeline-CodeCommit-CodeBuild/assets/91665833/e3547498-8608-4946-9638-1d1349d72d41)

 " terraform.tfstate" is created in S3

![image](https://github.com/felixdagnon/Devops-on-AWS-CICD-defined-through-Terraform-using-CodePipeline-CodeCommit-CodeBuild/assets/91665833/55b6f0f9-da31-4867-9459-d043d70eab27)

Terraform has been successfully initialized. 

Running command terraform validate, terraform plan, terraform apply

![image](https://github.com/felixdagnon/Devops-on-AWS-CICD-defined-through-Terraform-using-CodePipeline-CodeCommit-CodeBuild/assets/91665833/4739473c-c60d-4453-a336-e06d98a2f623)

terraform plan completed 

![image](https://github.com/felixdagnon/Devops-on-AWS-CICD-defined-through-Terraform-using-CodePipeline-CodeCommit-CodeBuild/assets/91665833/fbe6af95-19a8-43c8-b016-e9b4eb622017)

terraform apply completed 

![image](https://github.com/felixdagnon/Devops-on-AWS-CICD-defined-through-Terraform-using-CodePipeline-CodeCommit-CodeBuild/assets/91665833/56c48dcf-179c-470f-b393-e329b28d5660)

All ressources are completed in Dev

![image](https://github.com/felixdagnon/Devops-on-AWS-CICD-defined-through-Terraform-using-CodePipeline-CodeCommit-CodeBuild/assets/91665833/3d5c94aa-dd1b-477f-b449-b582fc610825)

0. Confirm SNS Subscription in your email

![image](https://github.com/felixdagnon/Devops-on-AWS-CICD-defined-through-Terraform-using-CodePipeline-CodeCommit-CodeBuild/assets/91665833/b96ba149-4717-495a-a2f5-8d1240505560)

Subscription confirmed

![image](https://github.com/felixdagnon/Devops-on-AWS-CICD-defined-through-Terraform-using-CodePipeline-CodeCommit-CodeBuild/assets/91665833/6ae9a114-86fb-44ef-90e5-b2614a184288)

SNS topic confirmed

![image](https://github.com/felixdagnon/Devops-on-AWS-CICD-defined-through-Terraform-using-CodePipeline-CodeCommit-CodeBuild/assets/91665833/95e2b97d-443a-46ed-90f5-c098731288e4)

1. Verify EC2 Instances

"dev-BastionHost" and  and 2 instances "hr-dev" are created and running

![image](https://github.com/felixdagnon/Devops-on-AWS-CICD-defined-through-Terraform-using-CodePipeline-CodeCommit-CodeBuild/assets/91665833/e2abc441-ef35-4878-8509-90c63247381b)

2. Verify Launch Templates (High Level)

It's created the lunch template name "hr-dev-2024022709342795200000000b" because we added "prefix"+local.name

![image](https://github.com/felixdagnon/Devops-on-AWS-CICD-defined-through-Terraform-using-CodePipeline-CodeCommit-CodeBuild/assets/91665833/24ad5629-d220-4909-b968-24d2874a7fd9)

3. Verify Autoscaling Group (High Level)

It's created the Autoscaling Group name "hr-dev-2024022709342795200000000b" because we added "prefix"+local.name

![image](https://github.com/felixdagnon/Devops-on-AWS-CICD-defined-through-Terraform-using-CodePipeline-CodeCommit-CodeBuild/assets/91665833/05ec8f91-36f9-4afd-a293-f737405a0127)

let's verify Target tracking policy

![image](https://github.com/felixdagnon/Devops-on-AWS-CICD-defined-through-Terraform-using-CodePipeline-CodeCommit-CodeBuild/assets/91665833/bef41afc-d4d2-4729-bdec-6626ec6eff8c)

4. Verify Load Balancer

Load Balancer name "hr-dev" created with listeners

![image](https://github.com/felixdagnon/Devops-on-AWS-CICD-defined-through-Terraform-using-CodePipeline-CodeCommit-CodeBuild/assets/91665833/8f09cbdb-7f42-4dbb-8097-75c908709be4)

Let's check certificate created

![image](https://github.com/felixdagnon/Devops-on-AWS-CICD-defined-through-Terraform-using-CodePipeline-CodeCommit-CodeBuild/assets/91665833/0021079b-af5b-481c-a117-60d4c73f2848)

5. Verify Load Balancer Target Group - Health Checks

Target groups are Healthy because the instance running in availabities zones are Healthy.

![image](https://github.com/felixdagnon/Devops-on-AWS-CICD-defined-through-Terraform-using-CodePipeline-CodeCommit-CodeBuild/assets/91665833/f8648b27-aca0-433e-b0bc-bfd87735d6f3)

6. Access and Test

```t
# Access and Test
http://devdemo5.kalyandemo.com
http://devdemo5.kalyandemo.com/app1/index.html
http://devdemo5.kalyandemo.com/app1/metadata.html
```

### Access and Test "http://devdemo5.kalyandemo.com"

Let's first verify Route 53. The record name "devdemo5.kalyandemo.com" created

![image](https://github.com/felixdagnon/Devops-on-AWS-CICD-defined-through-Terraform-using-CodePipeline-CodeCommit-CodeBuild/assets/91665833/7051c2c8-e246-40b2-8bcd-e7a07f6108e2)

Copy this link "devdemo5.kalyandemo.com" and paste url

![image](https://github.com/felixdagnon/Devops-on-AWS-CICD-defined-through-Terraform-using-CodePipeline-CodeCommit-CodeBuild/assets/91665833/a4e1d216-4541-4ffd-9110-672a5ce4c63a)

We obtain https connexion secure

![image](https://github.com/felixdagnon/Devops-on-AWS-CICD-defined-through-Terraform-using-CodePipeline-CodeCommit-CodeBuild/assets/91665833/e777edae-bb96-4b48-997f-1fbf30fcc3f7)

Click on to view the certificate complete information

![image](https://github.com/felixdagnon/Devops-on-AWS-CICD-defined-through-Terraform-using-CodePipeline-CodeCommit-CodeBuild/assets/91665833/50e7a6c5-eb19-4850-aeb2-6448a7339f2c)


### Access and Test "http://devdemo5.kalyandemo.com/app1/index.html"

The test is valid

![image](https://github.com/felixdagnon/Devops-on-AWS-CICD-defined-through-Terraform-using-CodePipeline-CodeCommit-CodeBuild/assets/91665833/dbd1567e-f953-4c69-8085-1da61e61c9da)

### Access and Test "http://devdemo5.kalyandemo.com/app1/metadata.html"

Matadata informations are completed. Two instances related private ip are refreshed. The load balancer is working

The first availably zone

![image](https://github.com/felixdagnon/Devops-on-AWS-CICD-defined-through-Terraform-using-CodePipeline-CodeCommit-CodeBuild/assets/91665833/33df182c-554d-4233-9392-49ec0e208df0)

The second availably zone

![image](https://github.com/felixdagnon/Devops-on-AWS-CICD-defined-through-Terraform-using-CodePipeline-CodeCommit-CodeBuild/assets/91665833/d7d8b6e7-e284-4cea-99e4-8a704e6572ab)

## Step-19: Add Approval Stage before deploying to staging environment

- Go to Services -> AWS CodePipeline -> tf-iacdevops-aws-cp1 -> Edit

![image](https://github.com/felixdagnon/Devops-on-AWS-CICD-defined-through-Terraform-using-CodePipeline-CodeCommit-CodeBuild/assets/91665833/f51169b8-1b81-45d4-a420-5b98d3cd7c8a)

### Add Stage

  - Name: Email-Approval
    
### Add Action Group

![image](https://github.com/felixdagnon/Devops-on-AWS-CICD-defined-through-Terraform-using-CodePipeline-CodeCommit-CodeBuild/assets/91665833/c7bee0f9-f103-4aec-b49e-516f502ab134)

- Action Name: Email-Approval

 ![image](https://github.com/felixdagnon/Devops-on-AWS-CICD-defined-through-Terraform-using-CodePipeline-CodeCommit-CodeBuild/assets/91665833/98081037-2ff9-4ebb-8f79-c2ae3515537c)

- Action Group

  ![image](https://github.com/felixdagnon/Devops-on-AWS-CICD-defined-through-Terraform-using-CodePipeline-CodeCommit-CodeBuild/assets/91665833/8222a25a-5d46-444a-a991-216b31bf11a0)

- Action Provider: Manual Approval
  
- SNS Topic: Select SNS Topic from drop down
  
- Comments: Approve to deploy to staging environment

![image](https://github.com/felixdagnon/Devops-on-AWS-CICD-defined-through-Terraform-using-CodePipeline-CodeCommit-CodeBuild/assets/91665833/2739d443-8fbd-41b1-8ba2-6e262d1d22ec)

## Step-20: Add Staging Environment Deploy Stage

- Go to Services -> AWS CodePipeline -> tf-iacdevops-aws-cp1 -> Edit
  
### Add Stage

![image](https://github.com/felixdagnon/Devops-on-AWS-CICD-defined-through-Terraform-using-CodePipeline-CodeCommit-CodeBuild/assets/91665833/a8ff40e2-7897-4400-993f-a8763461eee6)

  - Name: Stage-Deploy
    
![image](https://github.com/felixdagnon/Devops-on-AWS-CICD-defined-through-Terraform-using-CodePipeline-CodeCommit-CodeBuild/assets/91665833/0bcd8521-5a46-4b8d-a8b4-7569089cd307)

### Add Action Group

![image](https://github.com/felixdagnon/Devops-on-AWS-CICD-defined-through-Terraform-using-CodePipeline-CodeCommit-CodeBuild/assets/91665833/52aae07b-c004-40e2-b856-2baf41d32855)

- Action Name: Stage-Deploy
  
- Region: US East (N.Virginia)
  
- Action Provider: AWS CodeBuild
  
- Input Artifacts: Source Artifact

![image](https://github.com/felixdagnon/Devops-on-AWS-CICD-defined-through-Terraform-using-CodePipeline-CodeCommit-CodeBuild/assets/91665833/d56455b5-093f-43d1-8b52-d625f0225e17)
  
- **Project Name:** Click on **Create Project**
  
  - **Project Name:** stage-deploy-IACDEVOPS-CB
    
  - **Description:** CodeBuild Project for Staging Environment of IAC DevOps Terraform Demo

![image](https://github.com/felixdagnon/Devops-on-AWS-CICD-defined-through-Terraform-using-CodePipeline-CodeCommit-CodeBuild/assets/91665833/7ea09a55-c8ab-49ed-9b2f-02fb5e88447d)

  - **Environment image:** Managed Image
    
  - **Operating System:** Amazon Linux 2
    
  - **Runtimes:** Standard
    
  - **Image:** latest available today (aws/codebuild/amazonlinux2-x86_64-standard:3.0)
    
  - **Environment Type:** Linux

![image](https://github.com/felixdagnon/Devops-on-AWS-CICD-defined-through-Terraform-using-CodePipeline-CodeCommit-CodeBuild/assets/91665833/69b13e21-dd46-40e0-860d-69c75c923a26)
    
  - **Service Role:** New (leave to defaults including Role Name)
    
  - **Build specifications:** use a buildspec file
    
  - **Buildspec name - optional:** buildspec-stag.yml  (Ensure that this file is present in root folder of your github repository)

![image](https://github.com/felixdagnon/Devops-on-AWS-CICD-defined-through-Terraform-using-CodePipeline-CodeCommit-CodeBuild/assets/91665833/6cfc96a0-c9d7-4ec6-b068-6be05c3b14a9)
    
  - Rest all leave to defaults

![image](https://github.com/felixdagnon/Devops-on-AWS-CICD-defined-through-Terraform-using-CodePipeline-CodeCommit-CodeBuild/assets/91665833/d73c2c07-0617-4d43-aba0-6c6e12a62f6c)

  - Click on **Continue to CodePipeline**
    
- **Project Name:** This value should be auto-populated with `stage-deploy-tf-iacdevops-aws-cp1`
  
- **Build Type:** Single Build
  
- Click on **Done**

![image](https://github.com/felixdagnon/Devops-on-AWS-CICD-defined-through-Terraform-using-CodePipeline-CodeCommit-CodeBuild/assets/91665833/23fccad1-408e-4ded-8af3-9f98f836035c)

- Review Edit Action

![image](https://github.com/felixdagnon/Devops-on-AWS-CICD-defined-through-Terraform-using-CodePipeline-CodeCommit-CodeBuild/assets/91665833/0297f9ad-6dea-442a-95f6-f3dfeded2d6e)

- Click on **Save**

![image](https://github.com/felixdagnon/Devops-on-AWS-CICD-defined-through-Terraform-using-CodePipeline-CodeCommit-CodeBuild/assets/91665833/fde760f6-b7a1-4dbf-8d52-9648592fe93b)

- Now we add "Manual approval" and "Stage-Deploy"

![image](https://github.com/felixdagnon/Devops-on-AWS-CICD-defined-through-Terraform-using-CodePipeline-CodeCommit-CodeBuild/assets/91665833/04b684f2-b5ae-4a6c-a84c-47e36016fa87)

## Step-21: Update the IAM Role

Let's search this role "codebuild-stage-deploy-IACDEVOPS-CB-service-role" in IAM role service

![image](https://github.com/felixdagnon/Devops-on-AWS-CICD-defined-through-Terraform-using-CodePipeline-CodeCommit-CodeBuild/assets/91665833/a39011c8-a59e-4687-88df-3c483b8bfc37)

- Update the IAM Role created as part of this `stage-deploy-tf-iacdevops-aws-cp1` CodeBuild project by adding the policy `systems-manger-get-parameter-access1`

![image](https://github.com/felixdagnon/Devops-on-AWS-CICD-defined-through-Terraform-using-CodePipeline-CodeCommit-CodeBuild/assets/91665833/5d23d939-83df-43c2-9105-f3902f3f3e0c)
 
## Step-22: Run the Pipeline 

- Go to Services -> AWS CodePipeline -> tf-iacdevops-aws-cp1
  
- Click on **Release Change**

- Verify Source Stage
  
- Verify Build Stage (Dev Environment - Dev Depploy phase)
  
- Verify Manual Approval Stage - Approve the change

![image](https://github.com/felixdagnon/Devops-on-AWS-CICD-defined-through-Terraform-using-CodePipeline-CodeCommit-CodeBuild/assets/91665833/c4af7e03-85d1-46ac-a3ae-624983528e0f)

Let's go to email to approve 

![image](https://github.com/felixdagnon/Devops-on-AWS-CICD-defined-through-Terraform-using-CodePipeline-CodeCommit-CodeBuild/assets/91665833/334fb7cb-c8d7-4db3-a7b8-6f55b784a12b)

Let's consult email

![image](https://github.com/felixdagnon/Devops-on-AWS-CICD-defined-through-Terraform-using-CodePipeline-CodeCommit-CodeBuild/assets/91665833/0be0be8c-d308-4454-b373-1121576663f6)

## Review Approval Stage of pipeline

![image](https://github.com/felixdagnon/Devops-on-AWS-CICD-defined-through-Terraform-using-CodePipeline-CodeCommit-CodeBuild/assets/91665833/d820fc1e-7f12-4079-b708-38020c5673b7)

Review

![image](https://github.com/felixdagnon/Devops-on-AWS-CICD-defined-through-Terraform-using-CodePipeline-CodeCommit-CodeBuild/assets/91665833/5d26d058-b6db-4d07-8e19-38b5dc35297d)

Approval Stage of pipeline succeeded

![image](https://github.com/felixdagnon/Devops-on-AWS-CICD-defined-through-Terraform-using-CodePipeline-CodeCommit-CodeBuild/assets/91665833/9311bc3b-9f13-4795-aa2e-aee38df405d2)

- Verify Stage Deploy Stage
  
Let's check log event. 

![image](https://github.com/felixdagnon/Devops-on-AWS-CICD-defined-through-Terraform-using-CodePipeline-CodeCommit-CodeBuild/assets/91665833/6d654e70-8390-4343-acdb-7f5cfcb69a00)

Let's verify ressoures 

### VPC is created and inside all ressources

![image](https://github.com/felixdagnon/Devops-on-AWS-CICD-defined-through-Terraform-using-CodePipeline-CodeCommit-CodeBuild/assets/91665833/67da90e2-a3e2-4a85-a42a-6ee99b40c79d)

CodeBuild created underline ressourrces for VPC.

Let's verify codebuild log


It shows build and post build state succeeded

![image](https://github.com/felixdagnon/Devops-on-AWS-CICD-defined-through-Terraform-using-CodePipeline-CodeCommit-CodeBuild/assets/91665833/8c68c28d-21f3-4bd4-9453-73ca64962915)

  - Verify build logs

The entire ressources are completed

Download phase complete

![image](https://github.com/felixdagnon/Devops-on-AWS-CICD-defined-through-Terraform-using-CodePipeline-CodeCommit-CodeBuild/assets/91665833/744aafee-d960-40ec-8851-24537bf675d1)

Intall and pre build phases completed

![image](https://github.com/felixdagnon/Devops-on-AWS-CICD-defined-through-Terraform-using-CodePipeline-CodeCommit-CodeBuild/assets/91665833/46802df7-d751-487e-b21f-5eca9a00e0fd)

Initializing the backend...

Successfully configured the backend "s3"! Terraform will automatically use this backend.

![image](https://github.com/felixdagnon/Devops-on-AWS-CICD-defined-through-Terraform-using-CodePipeline-CodeCommit-CodeBuild/assets/91665833/f86bfee9-815b-41ec-b554-92815b07b1c8)

 " terraform.tfstate" is created in S3

![image](https://github.com/felixdagnon/Devops-on-AWS-CICD-defined-through-Terraform-using-CodePipeline-CodeCommit-CodeBuild/assets/91665833/4484a1de-5b29-4253-939f-897c579bbc28)

Terraform has been successfully initialized. 

Running command terraform validate, terraform plan, terraform apply

![image](https://github.com/felixdagnon/Devops-on-AWS-CICD-defined-through-Terraform-using-CodePipeline-CodeCommit-CodeBuild/assets/91665833/c6d6bcd5-c6da-45fb-ab48-8d05c0f5b581)

terraform plan completed 

![image](https://github.com/felixdagnon/Devops-on-AWS-CICD-defined-through-Terraform-using-CodePipeline-CodeCommit-CodeBuild/assets/91665833/ac8afb05-ab24-4f17-9ad6-0c73f7193d12)

terraform apply completed 

![image](https://github.com/felixdagnon/Devops-on-AWS-CICD-defined-through-Terraform-using-CodePipeline-CodeCommit-CodeBuild/assets/91665833/49af1c54-a15b-4e09-beca-f18a6c5fdf3b)

All ressources are completed in Stage environment

![image](https://github.com/felixdagnon/Devops-on-AWS-CICD-defined-through-Terraform-using-CodePipeline-CodeCommit-CodeBuild/assets/91665833/b2dc4a35-02cc-4d67-90ab-9039708c416c)

Pipeline looking

![image](https://github.com/felixdagnon/Devops-on-AWS-CICD-defined-through-Terraform-using-CodePipeline-CodeCommit-CodeBuild/assets/91665833/e6beabe8-c59d-41c0-8f45-f7075d4a40c5)

Source and build phases succeeded

![image](https://github.com/felixdagnon/Devops-on-AWS-CICD-defined-through-Terraform-using-CodePipeline-CodeCommit-CodeBuild/assets/91665833/59b06948-471d-4082-a1c0-6749ea453ffa)

Email-Approval and Stage-Deploy phases succeeded

![image](https://github.com/felixdagnon/Devops-on-AWS-CICD-defined-through-Terraform-using-CodePipeline-CodeCommit-CodeBuild/assets/91665833/fdb8f24e-cbd8-4b2d-8a04-9f0f530762a1)

## Step-23: Verify Staging Environment

0. Confirm SNS Subscription in your email

Notification by email

![image](https://github.com/felixdagnon/Devops-on-AWS-CICD-defined-through-Terraform-using-CodePipeline-CodeCommit-CodeBuild/assets/91665833/ea992a10-a0b4-4cfe-8185-7455984b0ad8)

Confirmation subscription

![image](https://github.com/felixdagnon/Devops-on-AWS-CICD-defined-through-Terraform-using-CodePipeline-CodeCommit-CodeBuild/assets/91665833/a4278aef-8e4f-417a-aeb2-6d1bbbeefbb0)

Let's verify SNS Topic

![image](https://github.com/felixdagnon/Devops-on-AWS-CICD-defined-through-Terraform-using-CodePipeline-CodeCommit-CodeBuild/assets/91665833/9f9ae445-b492-4570-b469-d5579eb46885)

1. Verify EC2 Instances

"stag-BastionHost" and and 2 instances "hr-stag" are created and running

![image](https://github.com/felixdagnon/Devops-on-AWS-CICD-defined-through-Terraform-using-CodePipeline-CodeCommit-CodeBuild/assets/91665833/92f5ea2d-5416-41e9-9e53-3f3b5786eacb)

2. Verify Launch Templates (High Level)

It's created the lunch template name "hr-stag-2024022810181677970000000c" because we added "prefix"+local.name

![image](https://github.com/felixdagnon/Devops-on-AWS-CICD-defined-through-Terraform-using-CodePipeline-CodeCommit-CodeBuild/assets/91665833/0ed0a522-729d-44b9-9528-f6bb2efc98cc)

3. Verify Autoscaling Group (High Level)

It's created the Autoscaling Group name "hr-stag-2024022810181768120000000e" because we added "prefix"+local.name

![image](https://github.com/felixdagnon/Devops-on-AWS-CICD-defined-through-Terraform-using-CodePipeline-CodeCommit-CodeBuild/assets/91665833/c2d47edf-3bf2-4748-9e8f-316e7331e1e1)

let's verify Target tracking policy

![image](https://github.com/felixdagnon/Devops-on-AWS-CICD-defined-through-Terraform-using-CodePipeline-CodeCommit-CodeBuild/assets/91665833/425d913d-cb09-4081-a3aa-ee57ac3483fb)

4. Verify Load Balancer

Load Balancer name "hr-stag" created with listeners

![image](https://github.com/felixdagnon/Devops-on-AWS-CICD-defined-through-Terraform-using-CodePipeline-CodeCommit-CodeBuild/assets/91665833/f00ce78a-cf34-4e99-8026-c227a6e232f8)

Let's check certificate created

![image](https://github.com/felixdagnon/Devops-on-AWS-CICD-defined-through-Terraform-using-CodePipeline-CodeCommit-CodeBuild/assets/91665833/8fce420b-1d65-4b06-b75e-1c87d2bc127b)

5. Verify Load Balancer Target Group - Health Checks

Target groups are Healthy because the instance running in availabities zones are Healthy.

![image](https://github.com/felixdagnon/Devops-on-AWS-CICD-defined-through-Terraform-using-CodePipeline-CodeCommit-CodeBuild/assets/91665833/a7673779-4015-45b0-8785-54869026a106)

6. Access and Test

```t
# Access and Test
http://stagedemo5.kalyandemo.com
http://stagedemo5.kalyandemo.com/app1/index.html
http://stagedemo5.kalyandemo.com/app1/metadata.html
```

### Access and Test "http://stagedemo5.kalyandemo.com"

Let's first verify Route 53. The record name "stagdemo5.kalyandemo.com" created

![image](https://github.com/felixdagnon/Devops-on-AWS-CICD-defined-through-Terraform-using-CodePipeline-CodeCommit-CodeBuild/assets/91665833/7b5a9245-2e2a-47e3-9a53-182d95dc09ea)

Copy this link "stagdemo5.kalyandemo.com" and paste url

![image](https://github.com/felixdagnon/Devops-on-AWS-CICD-defined-through-Terraform-using-CodePipeline-CodeCommit-CodeBuild/assets/91665833/466652fd-15b4-4262-933c-8e3bc4d384b6)

We obtain https connexion secure

![image](https://github.com/felixdagnon/Devops-on-AWS-CICD-defined-through-Terraform-using-CodePipeline-CodeCommit-CodeBuild/assets/91665833/f42fb884-0c1d-433b-85cc-78fc0d597bfe)

Click on to view the certificate complete information

![image](https://github.com/felixdagnon/Devops-on-AWS-CICD-defined-through-Terraform-using-CodePipeline-CodeCommit-CodeBuild/assets/91665833/71a7750a-4594-43d3-95e3-ef4d83788fc7)

### Access and Test "http://stagedemo5.kalyandemo.com/app1/index.html"

The test is valid

![image](https://github.com/felixdagnon/Devops-on-AWS-CICD-defined-through-Terraform-using-CodePipeline-CodeCommit-CodeBuild/assets/91665833/d3e96ef3-1e34-4db0-a9ec-e9bdb4ca534b)

### Access and Test "http://stagedemo5.kalyandemo.com/app1/metadata.html"

Matadata informations are completed. Two instances related private ip are refreshed. The load balancer is working

The first availably zone

![image](https://github.com/felixdagnon/Devops-on-AWS-CICD-defined-through-Terraform-using-CodePipeline-CodeCommit-CodeBuild/assets/91665833/d07ae1a2-9074-452e-952f-fb8865d2f864)

The second availably zone

![image](https://github.com/felixdagnon/Devops-on-AWS-CICD-defined-through-Terraform-using-CodePipeline-CodeCommit-CodeBuild/assets/91665833/52da223a-fd13-4361-afae-5554eaacbdce)

## Step-24: Make a change and test the entire pipeline

### Step-24-01: c13-03-autoscaling-resource.tf

- Increase minimum EC2 Instances from 2 to 3

```t
# Before
  desired_capacity = 2
  max_size = 10
  min_size = 2
# After
  desired_capacity = 4
  max_size = 10
  min_size = 4
```
![image](https://github.com/felixdagnon/Devops-on-AWS-CICD-defined-through-Terraform-using-CodePipeline-CodeCommit-CodeBuild/assets/91665833/14003e38-83e1-4e57-a36c-fa9eea9af18d)

### Step-24-02: Commit Changes via Git Repo
```t
# Verify Changes
git status
# Commit Changes to Local Repository
git add .
git commit -am "ASG Min Size from 2 to 4"

# Push changes to Remote Repository
git push
```
![image](https://github.com/felixdagnon/Devops-on-AWS-CICD-defined-through-Terraform-using-CodePipeline-CodeCommit-CodeBuild/assets/91665833/448c841d-56d9-4b48-8072-5f4cfd331d11)

### Step-24-03: Review Build Logs

- Go to Services -> CodePipeline -> tf-iacdevops-aws-cp1

Source and build phases succeeded

![image](https://github.com/felixdagnon/Devops-on-AWS-CICD-defined-through-Terraform-using-CodePipeline-CodeCommit-CodeBuild/assets/91665833/24c9fafa-fcdd-4c0d-ab62-b7ff3644d425)

- Verify Dev Deploy Logs

Autosclaling group modified in dev and increase capacity

![image](https://github.com/felixdagnon/Devops-on-AWS-CICD-defined-through-Terraform-using-CodePipeline-CodeCommit-CodeBuild/assets/91665833/9554a499-03d3-4175-b899-960f08bafe50)

- Approve at `Manual Approval` stage

![image](https://github.com/felixdagnon/Devops-on-AWS-CICD-defined-through-Terraform-using-CodePipeline-CodeCommit-CodeBuild/assets/91665833/62603d8d-a1fa-41be-8d14-2948f68cfd58)

Email-Approval and Stage-Deploy phases succeeded

![image](https://github.com/felixdagnon/Devops-on-AWS-CICD-defined-through-Terraform-using-CodePipeline-CodeCommit-CodeBuild/assets/91665833/7d78dbc1-6120-4e7b-98e1-5edddde1d43c)

  
- Verify Stage Deploy Logs

Autosclaling group modified in stage environement 2 ---> 4

![image](https://github.com/felixdagnon/Devops-on-AWS-CICD-defined-through-Terraform-using-CodePipeline-CodeCommit-CodeBuild/assets/91665833/6a0f2d15-5dd8-465c-8abc-9cb63580c73c)

Deployment complete succeeded

![image](https://github.com/felixdagnon/Devops-on-AWS-CICD-defined-through-Terraform-using-CodePipeline-CodeCommit-CodeBuild/assets/91665833/e2bf8676-9447-4946-83b5-ee8ef9efb3cc)

### Step-24-04: Verify EC2 Instances

- Go to Services -> EC2 Instances
  
- Newly created instances should be visible.
  
- hr-dev: 4 EC2 Instances

Let's verify instances

The number of instance increase to 4. 

![image](https://github.com/felixdagnon/Devops-on-AWS-CICD-defined-through-Terraform-using-CodePipeline-CodeCommit-CodeBuild/assets/91665833/c29d3174-210e-4c0d-9285-19197a7b83e8)

Let's check the target groups 

![image](https://github.com/felixdagnon/Devops-on-AWS-CICD-defined-through-Terraform-using-CodePipeline-CodeCommit-CodeBuild/assets/91665833/a6c6f166-56ba-48b5-8355-3a472818837b)

- hr-stag: 4 EC2 Instances

The number of instance increase to 4. 

![image](https://github.com/felixdagnon/Devops-on-AWS-CICD-defined-through-Terraform-using-CodePipeline-CodeCommit-CodeBuild/assets/91665833/aecd8b17-7e18-4cf0-b041-2a96d27946ff)

Let's check the target groups 

![image](https://github.com/felixdagnon/Devops-on-AWS-CICD-defined-through-Terraform-using-CodePipeline-CodeCommit-CodeBuild/assets/91665833/aadbe18b-65a8-4f44-a032-484be3e5f671)

let's verify autoscaling group

![image](https://github.com/felixdagnon/Devops-on-AWS-CICD-defined-through-Terraform-using-CodePipeline-CodeCommit-CodeBuild/assets/91665833/8d01cced-c99d-495b-a0fc-5228b43f3d6a)

let's verify autoscaling load balancer

![image](https://github.com/felixdagnon/Devops-on-AWS-CICD-defined-through-Terraform-using-CodePipeline-CodeCommit-CodeBuild/assets/91665833/b882a7dc-69ec-47a6-b24d-b9867724190a)

## Step-25: Destroy Resources

### Step-25-01: Update buildspec-dev.yml

```t
# Before
    TF_COMMAND: "apply"
    #TF_COMMAND: "destroy"
# After
    #TF_COMMAND: "apply"
    TF_COMMAND: "destroy"    
```

![image](https://github.com/felixdagnon/Devops-on-AWS-CICD-defined-through-Terraform-using-CodePipeline-CodeCommit-CodeBuild/assets/91665833/027a7c02-c98e-4cdb-a7ad-26d167f3dd25)

### Step-25-02: Update buildspec-stag.yml

```t
# Before
    TF_COMMAND: "apply"
    #TF_COMMAND: "destroy"
# After
    #TF_COMMAND: "apply"
    TF_COMMAND: "destroy"    
```

![image](https://github.com/felixdagnon/Devops-on-AWS-CICD-defined-through-Terraform-using-CodePipeline-CodeCommit-CodeBuild/assets/91665833/578bfca8-53be-4575-916a-c44c16f792fa)

### Step-25-03: Commit Changes via Git Repo

```t
# Verify Changes
git status

# Commit Changes to Local Repository
git add .
git commit -am "Destroy Resources"

# Push changes to Remote Repository
git push
```

![image](https://github.com/felixdagnon/Devops-on-AWS-CICD-defined-through-Terraform-using-CodePipeline-CodeCommit-CodeBuild/assets/91665833/57f3c2c7-226c-4267-b906-2eeeb14769cf)

### Step-25-03: Review Build Logs

- Go to Services -> CodePipeline -> tf-iacdevops-aws-cp1
  
- Verify Dev Deploy Logs

  All ressouces are destroying

![image](https://github.com/felixdagnon/Devops-on-AWS-CICD-defined-through-Terraform-using-CodePipeline-CodeCommit-CodeBuild/assets/91665833/06b63944-c08d-4e08-b179-f279ca8e1a39)

Let's verivy pipeline

![image](https://github.com/felixdagnon/Devops-on-AWS-CICD-defined-through-Terraform-using-CodePipeline-CodeCommit-CodeBuild/assets/91665833/d80a7971-c79f-4250-9a51-7eb1a19b11ea)
  
- Approve at `Manual Approval` stage

 ![image](https://github.com/felixdagnon/Devops-on-AWS-CICD-defined-through-Terraform-using-CodePipeline-CodeCommit-CodeBuild/assets/91665833/885eac8e-16bc-403f-8976-981641e98b1f)

Edit pipeline in approval stage

![image](https://github.com/felixdagnon/Devops-on-AWS-CICD-defined-through-Terraform-using-CodePipeline-CodeCommit-CodeBuild/assets/91665833/835d3a83-84cc-4d29-a706-ee991bb41041)

Change SNS Tipic

![image](https://github.com/felixdagnon/Devops-on-AWS-CICD-defined-through-Terraform-using-CodePipeline-CodeCommit-CodeBuild/assets/91665833/c8963260-0dcd-411a-98aa-bd7c9a97bff1)

Release pipeline

Reveiw Email-Approval stage and approve

![image](https://github.com/felixdagnon/Devops-on-AWS-CICD-defined-through-Terraform-using-CodePipeline-CodeCommit-CodeBuild/assets/91665833/504ba803-a1f5-448b-991b-eb459f920af0)

All ressources in Stage-Deploy are destroyed

![image](https://github.com/felixdagnon/Devops-on-AWS-CICD-defined-through-Terraform-using-CodePipeline-CodeCommit-CodeBuild/assets/91665833/3f38441b-53b2-4eac-9ec3-1f3e20b13fc8)

- Verify Stage Deploy Logs
  
![image](https://github.com/felixdagnon/Devops-on-AWS-CICD-defined-through-Terraform-using-CodePipeline-CodeCommit-CodeBuild/assets/91665833/b5aa4971-63e9-46f6-a681-fec6b6cc8ac6)

- Pipeline looking

![image](https://github.com/felixdagnon/Devops-on-AWS-CICD-defined-through-Terraform-using-CodePipeline-CodeCommit-CodeBuild/assets/91665833/7f8f1808-c8ff-4d2c-8cc9-471bbef4166a)

All phases completed

![image](https://github.com/felixdagnon/Devops-on-AWS-CICD-defined-through-Terraform-using-CodePipeline-CodeCommit-CodeBuild/assets/91665833/4ab9eb85-9c99-4c7c-9d05-17e86d042d01)

- Let's check ressources
  
All EC2 are terminated

![image](https://github.com/felixdagnon/Devops-on-AWS-CICD-defined-through-Terraform-using-CodePipeline-CodeCommit-CodeBuild/assets/91665833/5245f72f-af8d-471b-b3f6-d69e8d8ef0d7)

## Step-26: Change Everything back to original Demo State

### Step-26-01: c13-03-autoscaling-resource.tf

- Change them back to original state
  
```t
# Before
  desired_capacity = 4
  max_size = 10
  min_size = 4
# After
  desired_capacity = 2
  max_size = 10
  min_size = 2
```

### Step-26-02: buildspec-dev.yml and buildspec-stag.yml

- Change them back to original state
  
```t
# Before
    #TF_COMMAND: "apply"
    TF_COMMAND: "destroy"   
# After
    TF_COMMAND: "apply"
    #TF_COMMAND: "destroy"     
```
### Step-26-03: Commit Changes via Git Repo

```t
# Verify Changes
git status

# Commit Changes to Local Repository
git add .
git commit -am "Fixed all the changes back to demo state"

# Push changes to Remote Repository
git push
```




## References
- [1:Backend configuration Dynamic](https://www.terraform.io/docs/cli/commands/init.html)
- [2:Backend configuration Dynamic](https://www.terraform.io/docs/language/settings/backends/configuration.html#partial-configuration)
- [AWS CodeBuild Builspe file reference](https://docs.aws.amazon.com/codebuild/latest/userguide/build-spec-ref.html#build-spec.env)




