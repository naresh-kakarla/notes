# Terraform

Terraform is an open-source tool by HashiCorp for provisioning and managing infrastructure across various cloud providers (AWS, Azure, GCP, etc.) using declarative configuration files.

## Table of Contents
- [Install](#install)
- [Terraform Configuration Language (HCL)](#terraform-configuration-language-hcl)
- [Terraform Workflow](#terraform-workflow)
- [Terraform Project Structure](#terraform-project-structure)
- [Providers](#providers)
- [Resources](#resources)
- [Backend](#backend)
    - [How does the relation between S3 and DynamoDB work for Terraform state and locking?](#how-does-the-relation-between-s3-and-dynamodb-work-for-terraform-state-and-locking)
- []


## Install Terraform

```
brew tap hashicorp/tap
brew install hashicorp/tap/terraform

terraform -v
```

## Terraform Configuration Language (HCL)

```
provider "aws" {
  region = "us-east-1"
}

resource "aws_instance" "example" {
  ami           = "ami-0c55b159cbfafe1f0"
  instance_type = "t2.micro"
}
```

| Term         | Description                                              |
| ------------ | -------------------------------------------------------- |
| **Provider** | Plugin for a specific platform (AWS, Azure, etc.)        |
| **Resource** | Infrastructure component (EC2 instance, S3 bucket, etc.) |
| **Variable** | Input to make configurations flexible                    |
| **Output**   | Useful information displayed after deployment            |
| **State**    | Tracks infrastructure resources managed by Terraform     |


## Terraform Workflow

```
terraform init      # Initialize the project
terraform plan      # Show execution plan
terraform apply     # Apply the changes (provision infra)
terraform destroy   # Tear down the infra

```

## Terraform Project Structure

```
my-terraform-project/
├── main.tf
├── variables.tf
├── outputs.tf

```

**Example: Create an AWS EC2 instance**

**`main.tf`:**

```
provider "aws" {
  region = var.region
}

resource "aws_instance" "my_server" {
  ami           = var.ami
  instance_type = var.instance_type
}
```

**`variables.tf`:**

```
variable "region" {
  default = "us-east-1"
}
variable "ami" {
  default = "ami-0c55b159cbfafe1f0"
}
variable "instance_type" {
  default = "t2.micro"
}
```

**`outputs.tf`:**

```
output "instance_id" {
  value = aws_instance.my_server.id
}

```


## Providers

Providers are plugins in Terraform that allow it to interact with APIs of cloud platforms and other services. They act as a bridge between your Terraform configuration and the actual infrastructure you want to manage.

Let’s say you want to create infrastructure in AWS — you’d use the aws provider. For Azure, you’d use azurerm, and for Google Cloud, google.

**Example:**

```
provider "aws" {
  region  = "us-east-1"
  profile = "default"   # Optional: uses credentials from your local AWS CLI
}
```

**Common Providers**


| Provider   | Name in Terraform | Use Case                   |
| ---------- | ----------------- | -------------------------- |
| AWS        | `aws`             | EC2, S3, IAM, etc.         |
| Azure      | `azurerm`         | VMs, Storage, Networking   |
| GCP        | `google`          | GCE, GCS, IAM              |
| Kubernetes | `kubernetes`      | Deploying apps to clusters |
| GitHub     | `github`          | Repos, teams, webhooks     |
| Docker     | `docker`          | Containers and networks    |

**Multiple Providers & Aliases**

You can define multiple providers or even multiple configurations of the same provider using aliases:

**Example:**

```
provider "aws" {
  alias  = "east"
  region = "us-east-1"
}

provider "aws" {
  alias  = "west"
  region = "us-west-2"
}
```
Then in your resource:

```
resource "aws_instance" "web_east" {
  provider = aws.east
  ...
}
```

## Resources:

A resource in Terraform represents a piece of infrastructure you want to create, manage, or destroy — such as:

  - A virtual machine (EC2, Compute Engine)
  - A database instance
  - A storage bucket
  - A DNS record
  - A security group

You define resources using the resource block in your .tf files.

**Syntax**

```
resource "<PROVIDER>_<RESOURCE_TYPE>" "<NAME>" {
  # Configuration arguments go here
}
```

-  `PROVIDER`_`RESOURCE_TYPE`: Identifies what kind of resource (e.g., aws_instance, google_compute_instance)
-  `NAME`: A local name used to reference the resource within the Terraform config

**Example: Create an AWS EC2 Instance**

```
resource "aws_instance" "web_server" {
  ami           = "ami-0c55b159cbfafe1f0"
  instance_type = "t2.micro"

  tags = {
    Name = "MyWebServer"
  }
}
```

**Common Resource Types by Provider**

**AWS (aws):**

- `aws_instance` – EC2 virtual machine
- `aws_s3_bucket` – S3 bucket
- `aws_security_group` – Security group/firewall rules

**Azure (azurerm):**

- `azurerm_virtual_machine`
- `azurerm_resource_group`
- `azurerm_storage_account`

**Google Cloud (google):**

- `google_compute_instance`
- `google_storage_bucket`

**Reference Resources**

You can use one resource's output in another:

```
resource "aws_security_group" "web_sg" {
  # ...
}

resource "aws_instance" "web" {
  ami           = "ami-xxxxxx"
  instance_type = "t2.micro"
  vpc_security_group_ids = [aws_security_group.web_sg.id]
}

```

## Variables

Variables let you parameterize your Terraform code — instead of hardcoding values, you define inputs that can be customized when you run Terraform.
This helps you:
- Reuse configs across environments (dev, staging, prod)
- Avoid repetition
- Make your code cleaner and easier to manage

**Variable Declaration**

```
variable "instance_type" {
  description = "Type of AWS EC2 instance"
  type        = string
  default     = "t2.micro"   # optional, provides a fallback
}
```

**Using Variables in Configuration**
You access variables with the var. prefix:

```
resource "aws_instance" "web" {
  ami           = "ami-0c55b159cbfafe1f0"
  instance_type = var.instance_type

  tags = {
    Name = "MyWebServer"
  }
}

```

**How to Pass Variable Values**

| Method                                | Description                                                                   |
| ------------------------------------- | ----------------------------------------------------------------------------- |
| **Default** value in `variable` block | Used if no other value provided                                               |
| **Command line flag**                 | `terraform apply -var="instance_type=t3.small"`                               |
| **Terraform.tfvars file**             | Create a file like `terraform.tfvars` and define `instance_type = "t3.small"` |
| **Environment variables**             | Set environment variables like `TF_VAR_instance_type=t3.small`                |



**Example of terraform.tfvars**

```
instance_type = "t3.small"
```

Terraform automatically loads terraform.tfvars when running commands.

**Variable Types Supported**
- string
- number
- bool
- list(<type>)
- map(<type>)
- object({ ... })
- any

**Example: List Variable**
```
variable "availability_zones" {
  type    = list(string)
  default = ["us-east-1a", "us-east-1b"]
}

resource "aws_subnet" "example" {
  count = length(var.availability_zones)

  availability_zone = var.availability_zones[count.index]
  cidr_block        = "10.0.${count.index}.0/24"
  vpc_id            = aws_vpc.main.id
}


```

**Create variables.tf file**

- Create a file named variables.tf (the name is conventional, but you can name it anything as long as it ends with .tf).
- Define your variables inside it like this:

```
variable "instance_type" {
  description = "Type of AWS EC2 instance"
  type        = string
  default     = "t2.micro"
}

variable "environment" {
  description = "Environment name"
  type        = string
}

```





## Backend

A backend in Terraform determines where Terraform stores the state of your infrastructure and how operations like plan, apply, and refresh are executed.

By default, Terraform uses a local backend — it stores the terraform.tfstate file on your local machine.

**Why Is State Important?**

Terraform needs to track what resources it manages. This is done in a state file:

- What resources exist
- Their current settings
- Dependencies between them

Without the state, Terraform wouldn’t know what to change.


**Types of Backends**

| Backend                        | Description                                       |
| ------------------------------ | ------------------------------------------------- |
| **local**                      | Default; stores `terraform.tfstate` locally       |
| **S3**                         | Stores state remotely in AWS S3 (great for teams) |
| **Terraform Cloud/Enterprise** | Managed state storage + remote execution          |
| **GCS**                        | Google Cloud Storage bucket backend               |
| **AzureRM**                    | Stores state in Azure Blob Storage                |

**Example**

```
terraform {
  backend "s3" {
    bucket         = "my-terraform-state"
    key            = "env/dev/terraform.tfstate"
    region         = "us-east-1"
    dynamodb_table = "terraform-locks"  # Optional, for state locking
  }
}
```

Then run:

`terraform init`

Terraform will migrate your state to S3 if you've previously used a local backend.

**Benefits of Remote Backends**

| Feature                  | Benefit                                          |
| ------------------------ | ------------------------------------------------ |
| **Remote state storage** | Keeps state centralized and backed up            |
| **Collaboration**        | Enables teams to work on the same infra          |
| **State locking**        | Prevents race conditions via DynamoDB or similar |
| **Remote execution**     | Run plans/apply from Terraform Cloud securely    |



## How does the relation between S3 and DynamoDB work for Terraform state and locking?

**1. State File Storage: S3 Bucket**

  Terraform saves the state file (terraform.tfstate) in an S3 bucket. This file tracks your infrastructure.

**2. State Locking: DynamoDB Table**

  Because S3 can’t lock files, Terraform uses a DynamoDB table to manage locks.

**3. Lock Acquisition**

  Before changing the state, Terraform creates a lock record in DynamoDB with info like user and time.

  - If no lock exists, Terraform proceeds.
  - If a lock exists, Terraform waits or stops to avoid conflicts.

**4. Lock Release**

  When done, Terraform removes the lock record, allowing others to update the state.

**Summary**

| Component     | Role                                     |
| ------------- | ---------------------------------------- |
| **S3 Bucket** | Stores the **Terraform state file**      |
| **DynamoDB**  | Manages **locking to prevent conflicts** |



Ref# https://github.com/sidpalas/devops-directive-terraform-course/tree/main/05-language-features