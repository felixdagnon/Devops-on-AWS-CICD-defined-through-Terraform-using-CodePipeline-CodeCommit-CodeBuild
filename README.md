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


