# AWS CloudFormation — SAA-C03

Infrastructure as code (no resources manually created)

Think of AWS CloudFormation as infrastructure defined in code. 

Instead of manually creating a VPC, subnets, EC2 instances, security groups, and load balancers in the AWS Console, you describe the resources in a template. CloudFormation provisions and manages them for you.

Since you're learning ECS, imagine needing the same application infrastructure in development, staging, and production. CloudFormation helps you reproduce that infrastructure consistently.

## 1. How CloudFormation works

CloudFormation template
YAML or JSON describing AWS resources

CloudFormation
Creates, updates, and manages resources
The template is your infrastructure blueprint. CloudFormation interprets it and coordinates the creation of the resources.

## 2. The most important concepts

#### Template

The YAML or JSON file that describes the infrastructure you want. It can define resource types, configuration, parameters, outputs, and dependencies.

#### Stack

A collection of AWS resources managed together through CloudFormation. Creating a stack provisions its resources; updating the stack applies template changes.

#### Change set

A preview of the changes CloudFormation expects to make to a stack before you execute an update.

#### Drift detection

Checks whether supported resource configurations have changed outside CloudFormation compared with the expected template configuration.

### Template vs. stack

Remember this distinction:

- Template = recipe.
- Stack = infrastructure created and managed using that recipe.

You can use the same template to create separate stacks for development and production, with different parameter values.

###### You can visualize CloudFormation using Infrastructure Composer
This is a visualizer for the flow of your CloudFormation. example: a wordpress cloudformation stack, how different services connect to each other

## 3. What does a template look like?

Here is a simplified example that defines an S3 bucket:

```

AWSTemplateFormatVersion: '2010-09-09'
Description: A simple S3 bucket

Resources:
  ApplicationBucket:
    Type: AWS::S3::Bucket
    Properties:
      BucketEncryption:
        ServerSideEncryptionConfiguration:
          - ServerSideEncryptionByDefault:
              SSEAlgorithm: AES256

```

When you deploy this template, CloudFormation creates an S3 bucket with server-side encryption configured.

Notice the structure:
- `Resources` lists the infrastructure to create.
- `Type` specifies the AWS resource type.
- `Properties` configures that resource.

This example does not specify a bucket name, so CloudFormation generates one. S3 bucket names must be globally unique.

### Service Role
These are IAM roles that allow CloudFormation create/update/delete stack resources on your behalf
Also gives users ability to carry out these create/update/delete actions to the stack resources even if they don't have permissions to work with the resources in the stack
 
 The user must have the permission "iam:PassRole"

## 4. Why use CloudFormation?

Suppose you need to create the same architecture in three environments.

Without infrastructure as code, you might manually create each resource and accidentally configure one security group differently.

With CloudFormation:

1. Define the architecture once.
2. Deploy a stack for each environment.
3. Change parameters to customize each environment.
4. Update the template when the architecture changes.
5. Review potentially disruptive changes using a change set.

CloudFormation helps improve consistency and repeatability. It does not automatically guarantee that the template is secure or that every deployment will succeed.

## 5. CloudFormation vs. related AWS services

| Service           | Primary job                                                                                 |
| ----------------- | ------------------------------------------------------------------------------------------- |
| CloudFormation    | Provision and manage AWS infrastructure from templates                                      |
| AWS CDK           | Define infrastructure using programming languages, then synthesize CloudFormation templates |
| Elastic Beanstalk | Simplify application deployment and environment management                                  |
| Systems Manager   | Manage and operate resources, including configuration and automation                        |
| Terraform         | Infrastructure as code across AWS and other supported providers                             |

Exam clue: If a question asks how to create and manage a repeatable collection of AWS resources using declarative YAML or JSON, choose CloudFormation

---
### Terraform
This is an Infrastructure as code (IaC) tool that lets you define and manage cloud infrastructure using config files instead of manually creating resources in AWS Console
It uses HashiCorp Configuration Language (HCL). 

---

## 6. Exam traps you should know

- Stacks are regional in practice: Most resource deployments are scoped to a Region. CloudFormation supports cross-Region deployments through mechanisms such as stack sets and nested or coordinated deployments.
- Dependencies: CloudFormation can infer many dependencies from resource references, such as `Ref` and `Fn::GetAtt`. Use `DependsOn` when an explicit dependency is needed.
- Rollback: If stack creation fails, CloudFormation generally attempts to roll back the operation according to the configured behavior.
- Deletion: Deleting a stack can delete its resources. `DeletionPolicy: Retain` can preserve a resource when the stack is deleted.
- Updates can replace resources: Changing certain properties requires resource replacement, which can cause disruption or data loss if you don't plan carefully.
- Drift detection isn't continuous prevention: It checks for configuration differences; it does not automatically stop someone from changing resources manually.

## 7. CloudFormation vs. Elastic Beanstalk: an important distinction

You might remember that Elastic Beanstalk provisions resources such as EC2 instances, load balancers, and Auto Scaling groups.

CloudFormation can also create those resources, but it operates at a different level:

- Elastic Beanstalk: Focuses on deploying and operating an application environment.
- CloudFormation: Focuses on defining and managing the underlying infrastructure as a collection of resources.

CloudFormation can be used to provision infrastructure for an application that you subsequently deploy through another service.