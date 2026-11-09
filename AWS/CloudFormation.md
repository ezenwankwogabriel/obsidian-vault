# AWS CloudFormation — SAA-C03

Infrastructure as code (IaC) (no resources manually created)

Think of AWS CloudFormation as infrastructure defined in code. 

Instead of manually creating AWS Resources like a VPC, subnets, EC2 instances, security groups, and load balancers in the AWS Console, you describe the resources in a template. CloudFormation provisions and manages them for you.

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

### How does CloudFormation know what to create first?
CloudFormation analyses dependencies between resources and creates them in a valid order.

Suppose you're creating an EC2 instance that needs a security group, The group must exist before the instance can reference it. 

### How does CloudFormation know that a task is done and ready for next
- CreationPolicy: Tells CloudFormation to wait for a specified number of success signals
- cfn-signal: A helper script that sends a success or failure signal to CloudFormation

The creation policy is invoked only when AWS CloudFormation creates the associated resource. Currently, the only AWS CloudFormation resources that support creation policies are `AWS::AutoScaling::AutoScalingGroup`, `AWS::EC2::Instance`, and `AWS::CloudFormation::WaitCondition`.

Use the **CreationPolicy** attribute when you want to **wait** on resource configuration actions before stack creation proceeds. For example, if you install and configure software applications on an EC2 instance, you might want those applications to be running before proceeding. In such cases, you can add a `CreationPolicy` attribute to the instance and then send a success signal to the instance after the applications are installed and configured.

The instance sends the success signal only after its configuration script has finished successfully.

```
Resources:
  ActiveDirectory:
    Type: AWS::EC2::Instance

    CreationPolicy:
      ResourceSignal:
        Count: 1
        Timeout: PT45M
```
This tells CloudFormation:
- Expect one success signal
- Wait for up to 45 minutes for it

Inside the EC2 instance, your configuration script might execute:
```
# Install and configure Active Directory
Install-WindowsFeature AD-Domain-Services

# Perform domain controller configuration
# ...

# Signal success only after configuration succeeds
cfn-signal.exe -e $LASTEXITCODE `
  --stack SharePointStack `
  --resource ActiveDirectory `
  --region us-east-1
```

Other important concepts in the CloudFormation 

| CloudFormation feature | What it guarantees                                                                              |
| ---------------------- | ----------------------------------------------------------------------------------------------- |
| `DependsOn`            | Resource creation order (create dependent first)                                                |
| `CreationPolicy`       | Waits for successful signals before marking a resource created                                  |
| `cfn-signal`           | Sends the success/failure signal                                                                |
| `WaitCondition`        | Pauses stack progress until an external signal arrives                                          |
| UpdatePolicy           | Controls how certain resources are updated                                                      |
| DeletionPolicy         | Controls what happens to a resource after it's removed from a stack / the stack is deleted      |
| UpdateReplacePolicy    | Controls what happens to the old physical resource when a stack update requires its replacement |
Also Remember:

| Helper     | Purpose                                                                  |
| ---------- | ------------------------------------------------------------------------ |
| UserData   | Runs boot-time scripts(a script passed to the instance when it launches) |
| cfn-init   | Configures EC2 using template metadata                                   |
| cfn-signal | Reports success or failure                                               |
| cfn-hup    | Detects changes to instance metadata and runs configured hooks           |

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
# Circular deps in CloudFormation
A Circular dependency happens when two or more resources depend on each other in a way that makes cloudFormation unable to determine which resource to create first.

If Resource A needs Resource B to exist first, and Resource B also needs A to exist first, neither can be created

Example: Imagine you have two resources
- An EC2 instance
- A security group
You want the EC2 instance to use the security group.
But suppose you also configure the security group to reference the EC2 instances private IP in one of its ingress rule
```
EC2 Instance
    |
    | Requires
    v
Security Group
    |
    | Requires EC2's private IP
    v
EC2 Instance
```

A ref/Sub is used in this case to refer to the entity being referend

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