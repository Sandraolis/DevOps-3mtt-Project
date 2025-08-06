# Terraform Modules - VPC and S3 Bucket with Backend Storage

## Purpose

In this mini project, I will use Terraform to create modularized configurations for building and Amazon Virtual Private Cloud (VPC) and an Amazon S3 bucket. Additionally, I will configure Terraform to use Amazon S3 as the backend storage for storing the Terraform state file.

##  Objectives

1. **Terraform Modules**:
- Learn how to create and use Terraform modules for modular infrastructure provisioning.
2. **VPC Creation:**
- Build a reusable Terraform module for creating a VPC with specfied configurations.
3. **S3 Bucket creation**
- Develop a Terraform module for creating an S3 bucket with customizable settings
4. **Backend Storage Configuration:**
- Configure Terraform to use `Amazon S3` as the backend storage for storing the Terraform state file.


# Project Tasks

## Task 1:  VPC Module
1. Create a new directory for my Terraform project `terraform-modules-vpc-s3`.

2. Inside the project directory, create a directory for the VPC module `modules/vpc`.

![](./Images/1.%20directory.png)


3. Write a Terraform module which I name the file main.tf for creating VPC with customizable configurations such as CIDR block, subnets, etc.

``` bash
resource "aws_vpc" "main" {
  cidr_block = var.cidr_block

  tags = {
    Name = var.vpc_name
  }
}
```

4. Still inside the same directory I create another file which will serves as my variable file for VPC creation.

``` bash
vi variables.tf
```

``` bash
variable "cidr_block" {
  description = "CIDR block for the VPC"
  type        = string
}

variable "vpc_name" {
  description = "Name tag for the VPC"
  type        = string
}
```


![](./Images/3.%20variables.tf.png)


# Task 2: S3 Bucket Module

1. Inside the project directory, create another directory for the S3 bucket module `modules/s3.`

2. Write a Terraform module `modules/s3/main.tf` for creating an S3 bucket with customizable configuration such as bucket name, ACL, etc.

3. Modify the main Terraform configuration file `main.tf` to use the S3 module and create an S3 bucket.

``` bash
resource "aws_s3_bucket" "this" {
  bucket = var.bucket_name
  acl    = var.acl

  tags = {
    Name = var.bucket_name
  }
}
```

4. Also create varibles file for S3 bucket


``` bash
vi variables
```

``` bash
variable "bucket_name" {
  description = "The name of the S3 bucket"
  type        = string
}

variable "acl" {
  description = "The canned ACL to apply"
  type        = string
}
```

![](./Images/4.%20s3.png)

![](./Images/5.%20variables.png)


# Task 3: Backend Storage Configuration

1. Configure Terraform to use Amazon S3 as the backend storage for storing the Terraform state file.

2. Create a backend configuration file `backend.tf` specifying the S3 bucket and key storing state.

3. Initialize the Terraform project using the command.


``` bash
terraform {
  backend "s3" {
    bucket         = "your-terraform-state-bucket"
    key            = "terraform.tfstate"
    region         = "us-east-1"
    encrypt        = true
    dynamodb_table = "your-lock-table"
  }
}
```


# Task 4: Root files

So at the root I create the folloewing file

1. backend.tf

``` bash
vi backend.tf
```

``` bash
terraform {
  backend "s3" {
    bucket         = "terraform-state-iyanu-20250722"
    key            = "terraform.tfstate"
    region         = "us-east-1"
    encrypt        = true
    dynamodb_table = "your-lock-table"
  }
}
```

2. provider.tf

- This is where I specify the provider I want to use for this project which is `aws`.


``` bash
vi provider.tf
```

``` bash
---
provider "aws" {
  region = var.aws_region
}
```

3. main.tf

This is my main configuration

``` bash
vi main.tf
```

``` bash
---
module "vpc" {
  source     = "./modules/vpc"
  cidr_block = var.vpc_cidr
  vpc_name   = var.vpc_name
}

module "s3_bucket" {
  source      = "./modules/s3"
  bucket_name = var.bucket_name
  acl         = var.bucket_acl
}
```

4. variables.tf

This is the variables I will pass to other file configurations.


``` bash
vi variables.tf
```

``` bash
variable "aws_region" {
  description = "AWS region"
  type        = string
}

variable "vpc_cidr" {
  description = "CIDR block for the VPC"
  type        = string
}

variable "vpc_name" {
  description = "Name tag for the VPC"
  type        = string
}

variable "bucket_name" {
  description = "S3 bucket name"
  type        = string
}

variable "bucket_acl" {
  description = "ACL for the S3 bucket"
  type        = string
  default     = "private"
}
```

5. terraform.tfvars

This is where I will be defining all the variable which will make it easy for other people to edit and use my configurations/ script.

``` bash
vi terraform.tfvars
```

``` bash
aws_region  = "us-east-1"
vpc_cidr    = "10.0.0.0/16"
vpc_name    = "MyCustomVPC"
bucket_name = "my-unique-bucket-name-12345"
bucket_acl  = "private"
```

![](./Images/6.%20cat-files.png)

![](./Images/7.%20terraform.tfvars.png)


# How to apply

1. After getting all my script ready I ran this command.

``` bash
terraform init
```

![](./Images/8.%20terraform-init.png)


``` bash
terraform validate

terraform plan
```

![](./Images/9.%20Terraform%20Validate%20and%20Plan.png)


``` bash
terraform apply
```

![](./Images/10.%20Terraform%20Apply.png)


2. Check the output on the aws portal.

![](./Images/11.%20terraform%20S3%20Bucket.png)

![](./Images/12.%20Terraform%20VPC.png)







