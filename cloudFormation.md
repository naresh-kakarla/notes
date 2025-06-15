# Cloud Formation

## Table of Contents

- [What is AWS CloudFormation?](#what-is-aws-cloudformation)
- [Why Use CloudFormation?](#why-use-cloudformation)
- [CloudFormation Template Basics](#cloudformation-template-basics)
- [Sample CloudFormation Template](#sample-cloudformation-template)
- [Intrinsic Functions](#intrinsic-functions)
- [CloudFormation Template Sections](#cloudformation-template-sections)
- [Deploy CloudFormation Templates](#deploy-cloudformation-templates)
- [AWS CLI Commands](#aws-cli-commands)
- [Best Practices](#best-practices)
- [References](#references)


## What is AWS CloudFormation?

AWS CloudFormation is a service that helps you model, provision, and manage your AWS infrastructure using code.

Instead of manually setting up your AWS resources (like EC2 instances, S3 buckets, or databases) through the AWS Console, CloudFormation allows you to define them in a text file called a template (in YAML or JSON format). This template acts as a blueprint for your cloud environment.

This approach is known as Infrastructure as Code (IaC).

## Why Use CloudFormation?

- **Automation**: Automate resource creation and updates — no manual clicks.

- **Consistency**: Deploy the same environment repeatedly with zero differences.

- **Version Control**: Store your infrastructure templates in Git for easy tracking and rollback.

- **Rollback on Errors**: If deployment fails, CloudFormation rolls back changes automatically.

- **Manage Complex Infrastructure**: Manage hundreds of resources as a single unit called a "stack."



## CloudFormation Template Basics

A CloudFormation template is a plain text file written in YAML or JSON that describes your AWS resources and their configurations.

- You define what resources you want, and CloudFormation figures out how to create them.
- You can include parameters to customize your template when deploying.
- You can define outputs to export useful information like instance IDs or IP addresses after deployment.



## Sample CloudFormation Template

```
AWSTemplateFormatVersion: "2010-09-09"
Description: >
  Simple EC2 instance template to launch a t2.micro instance.

Parameters:
  InstanceType:
    Description: EC2 instance type
    Type: String
    Default: t2.micro
    AllowedValues:
      - t2.micro
      - t2.small
      - t3.micro
    ConstraintDescription: must be a valid EC2 instance type.

Resources:
  MyEC2Instance:
    Type: AWS::EC2::Instance
    Properties:
      InstanceType: !Ref InstanceType
      ImageId: ami-0abcdef1234567890  # Replace with a valid AMI in your region
      Tags:
        - Key: Name
          Value: CloudFormationInstance

Outputs:
  InstanceId:
    Description: "The Instance ID"
    Value: !Ref MyEC2Instance

  PublicIP:
    Description: "Public IP address of the instance"
    Value: !GetAtt MyEC2Instance.PublicIp
```

## Intrinsic Functions

Intrinsic functions are built-in commands in CloudFormation templates that allow dynamic references to resource properties, substitution of variables, joining strings, and importing values from other stacks, making templates flexible and reusable.

| Function       | Purpose                                             | Example Usage                       | Output Example                                     |
| -------------- | --------------------------------------------------- | ----------------------------------- | -------------------------------------------------- |
| `!Ref`         | Returns a resource's physical ID or parameter value | `!Ref MyBucket`                     | `my-bucket-name` (S3 bucket name)                  |
| `!GetAtt`      | Gets a specific attribute from a resource           | `!GetAtt MyInstance.PublicIp`       | `192.0.2.0` (EC2 instance public IP address)       |
| `!Sub`         | Substitutes variables into a string                 | `!Sub 'arn:aws:s3:::${MyBucket}'`   | `arn:aws:s3:::my-bucket-name`                      |
| `!Join`        | Joins strings together                              | `!Join [":", ["arn", "aws", "s3"]]` | `arn:aws:s3`                                       |
| `!ImportValue` | Imports output from another stack                   | `!ImportValue SharedVpcId`          | `vpc-0a1b2c3d4e5f6g7h` (VPC ID from another stack) |


**Examples:**

```
# CloudFormation Intrinsic Functions Summary

# 1. !Ref
# Purpose: Returns a resource's physical ID or parameter value.
# Example Usage: !Ref MyBucket
# Output Example: "my-bucket-name" (S3 bucket name)

Resources:
  MyBucket:
    Type: AWS::S3::Bucket

Outputs:
  BucketName:
    Description: "The name of the S3 bucket"
    Value: !Ref MyBucket


# 2. !GetAtt
# Purpose: Gets a specific attribute from a resource.
# Example Usage: !GetAtt MyInstance.PublicIp
# Output Example: "192.0.2.0" (EC2 instance public IP address)

Resources:
  MyInstance:
    Type: AWS::EC2::Instance
    Properties:
      ImageId: ami-0abcdef1234567890
      InstanceType: t2.micro

Outputs:
  InstancePublicIP:
    Description: "Public IP address of the EC2 instance"
    Value: !GetAtt MyInstance.PublicIp


# 3. !Sub
# Purpose: Substitutes variables into a string.
# Example Usage: !Sub 'arn:aws:s3:::${MyBucket}'
# Output Example: "arn:aws:s3:::my-bucket-name"

Resources:
  MyBucket:
    Type: AWS::S3::Bucket

Outputs:
  BucketArn:
    Description: "The ARN of the S3 bucket"
    Value: !Sub 'arn:aws:s3:::${MyBucket}'


# 4. !Join
# Purpose: Joins strings together.
# Example Usage: !Join [":", ["arn", "aws", "s3"]]
# Output Example: "arn:aws:s3"

Outputs:
  ServiceArn:
    Description: "ARN constructed using Join"
    Value: !Join 
      - ":" 
      - - "arn"
        - "aws"
        - "s3"


# 5. !ImportValue
# Purpose: Imports output exported from another CloudFormation stack.
# Example Usage: !ImportValue SharedVpcId
# Output Example: "vpc-0a1b2c3d4e5f6g7h" (VPC ID from another stack)

Outputs:
  VpcId:
    Description: "Import VPC ID from another stack"
    Value: !ImportValue SharedVpcId

# 6. !Equal
# Purpose: Compares two values and returns true if they are equal, otherwise false.
# Example Usage: !Equal [ "Value1", "Value2" ]
# Output Example: true or false

# Usage in Conditions section (common use case):
Conditions:
  IsProd: !Equal [ !Ref EnvType, "prod" ]

Outputs:
  IsProdEnvironment:
    Description: "Whether the environment is production"
    Value: !If 
      - IsProd
      - "Yes"
      - "No"

```

## CloudFormation Template Sections

| Section                      | Description                                                                        | Required? |
| ---------------------------- | ---------------------------------------------------------------------------------- | --------- |
| **AWSTemplateFormatVersion** | (Optional) Version of the CloudFormation template format (usually `"2010-09-09"`). | No        |
| **Description**              | (Optional) A human-readable description of the template.                           | No        |
| **Parameters**               | (Optional) Input values you can pass when launching a stack, e.g., instance size.  | No        |
| **Mappings**                 | (Optional) Fixed mappings for static values like region-to-AMI IDs.                | No        |
| **Conditions**               | (Optional) Logic to control resource creation under specific conditions.           | No        |
| **Resources**                | (Required) AWS resources to create like EC2, S3, RDS, etc.                         | Yes       |
| **Outputs**                  | (Optional) Useful information returned after the stack is created.                 | No        |

**Example:**

```
AWSTemplateFormatVersion: '2010-09-09'  # Optional: Template format version

Description: >
  Simple example template demonstrating all main CloudFormation sections.

Parameters:                            # Optional: Input parameters to customize the stack
  EnvType:
    Description: "Environment type (dev or prod)"
    Type: String
    Default: dev
    AllowedValues:
      - dev
      - prod

Mappings:                             # Optional: Static mappings for region to AMI IDs
  RegionMap:
    us-east-1:
      AMI: ami-0abcdef1234567890
    us-west-2:
      AMI: ami-0123456789abcdef0

Conditions:                          # Optional: Conditions for resource creation logic
  IsProd: !Equal [ !Ref EnvType, prod ]

Resources:                          # Required: Define AWS resources to create
  MyBucket:
    Type: AWS::S3::Bucket
    Properties:
      BucketName: !Sub "my-app-${EnvType}-bucket"   # Bucket name uses parameter substitution

  MyInstance:
    Type: AWS::EC2::Instance
    Condition: IsProd             # Only create EC2 if environment is prod
    Properties:
      InstanceType: t2.micro
      ImageId: !FindInMap [RegionMap, !Ref "AWS::Region", AMI]

Outputs:                           # Optional: Outputs after stack creation
  BucketNameOutput:
    Description: "Name of the S3 bucket"
    Value: !Ref MyBucket

  InstanceIdOutput:
    Description: "ID of the EC2 instance (only in prod)"
    Value: !Ref MyInstance
    Condition: IsProd
```

## Deploy CloudFormation Templates

**Using Console**

1. Go to AWS CloudFormation service.
2. Click Create Stack → With new resources (standard).
3. Upload your template file or paste it directly.
4. Provide stack name and parameters (like InstanceType).
5. Review and create the stack.
6. Watch stack events until creation is complete.

**Using AWS CLI**

```
aws cloudformation create-stack \
  --stack-name MyFirstStack \
  --template-body file://template.yaml \
  --parameters ParameterKey=InstanceType,ParameterValue=t2.micro
```

To check stack creation status:

```
aws cloudformation describe-stacks --stack-name MyEC2Stack
```



## AWS CLI Commands
1. Create Stack

```
aws cloudformation create-stack \
  --stack-name MyStack \
  --template-body file://template.yaml

```

2. Update Stack

```
aws cloudformation update-stack \
  --stack-name MyStack \
  --template-body file://template.yaml
```

3. Delete Stack
```
aws cloudformation delete-stack \
  --stack-name MyStack
```


4. Validate Template

```
aws cloudformation validate-template \
  --template-body file://template.yaml
```

5. Get stack info	

```
aws cloudformation describe-stacks --stack-name MyStack
```


## Best Practices
- Use Parameters: Make templates reusable and customizable.
- Use Outputs: Export important information for other stacks or users.
- Version control: Store your templates in Git.
- Use modular design: Break large templates into nested stacks.
- Test in Dev first: Always test your templates in a non-production environment.
- Enable rollback: Use automatic rollback on failure to avoid partial deployments.
- Use IAM Roles and Least Privilege: Control permissions tightly.
- Document your templates: Use the Description section clearly.


## References

https://github.com/aws-cloudformation/aws-cloudformation-templates