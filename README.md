# Devops-on-AWS-CICD-defined-through-Terraform-using-CodePipeline-CodeCommit-CodeBuild
Devops on AWS CI/CD defined through Terraform using CodePipeline, CodeCommit CodeBuild


Before delving into details, let’s first take a look at the picture.

![image](https://github.com/felixdagnon/Devops-on-AWS-CICD-defined-through-Terraform-using-CodePipeline-CodeCommit-CodeBuild/assets/91665833/0c28eda5-ec5a-4642-afda-59561d69e894)


As far as code functionality, my piece handles the following requirements:

- a way to provision Permission Sets into the AWS Account that hosts the AWS SSO service;

- a way to link these Permission Sets to Groups and AWS Accounts;

- a CI/CD to deploy the Permission Sets automatically when a new set or Group/Account link is configured
  
and pushed to the code repository;

- documentation of the solutions;

- the code translates into infrastructure which will be built by running terraform commands.

- The environment that is being built will be used to vend Accounts in AWS from a Management account that relies
  
 on AWS SSO to allow user access. Users assume Roles (shaped as Permission Sets) assigned to Groups that they are
 
 part of, which, in turn, are assigned to AWS Accounts.


Permission Sets dictate the level of access a User has to the many of available services.

Different teams will have different Permission Sets assigned for different Accounts they need access to and all of their Users will access

AWS through one AWS SSO portal which will be available through a URL link (this and many other components are provisioned separately).

The challenge comes from making this process a repeatable one with as little human interaction as possible. While I’m writing this post, 

AWS has not yet added any API functionality for the SSO service to allow provisioning Users and Groups programmatically through calls, 

so this step needs to be performed from the AWS SSO Console, but the rest of the steps can be done programmatically through either the

AWS cli or through terraform code.

Let focus on CI/CD components from the diagram above.

![image](https://github.com/felixdagnon/Devops-on-AWS-CICD-defined-through-Terraform-using-CodePipeline-CodeCommit-CodeBuild/assets/91665833/021157b0-7bd9-418f-aaee-891be1196b0a)

 AWS provides solutions for all of my needs:

- AWS CodeCommit acts as a git server, providing repositories to store code and allows interaction with your code through the git cli;

- AWS CodeBuild acts as a Build environment/engine and is used to execute instructions in multiple stages, to build and pack code in the

shape of a usable artifact;

- AWS CodePipeline is the CI/CD framework that links the other two services together (as well as others) through executable Stages;

- AWS S3 to keep any artifacts that result out of a successful Build Stage, for later use and posterity.



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
  
For critical projects which you want to isolate as above, Terraform also recommends this approach but its all case to case basis on the environment we have built, skill level and 

organization level standards.




## Step-03-02: Option-2: Create only 1 folder and leverage same C1 to C13 files (approx 30 files) across environments.

Only 30 files to manage across Dev, QA, Staging, Production and DR environments.

We are going to take this option-2 and build the pipeline for Dev and Staging environments




  



