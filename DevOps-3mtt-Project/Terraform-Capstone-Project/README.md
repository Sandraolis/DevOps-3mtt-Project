# Terraform Capstone Project: Automated WordPress Deployment on AWS

# Project Scenario

DigitalBoost, a digital marketing agency, aim to elevate its online presence by launching a high-performance WordPress Website for their clients. As an AWS Solution Architect, my task is to design and implement a scalable, secure, and cost-effective WordPress solutions using various AWS services.

Automating through Terraform is the key to achieve a streamlined and reproducible deployment process.

# Project Components

1. VPC Setup

- `Objective`: Create a Virtual Private Cloud (VPC) to isolate and secure the WordPress infrastructure.

- `Steps:`

1. Define IP address range for the VPC.
2. Create VPC with public and private subnets.
3. Configure route tables for each subnets.

# Terraform Configuration

1. I created a new directory for this project which is `terraform-wordpress`. and i navigated into it.

``` bash
mkdir terraform-wordpress

cd terraform-wordpress
```

2. Then create another directory for VPC.

``` bash
mkdir -p modules/vpc

cd modules/vpc
```

3. Then I created the following files which is needed for this project, e.g. `main.tf`, `variables.tf` and `outputs.tf`
note: make sure you grant any files RWE permissions, if the file refuses to save

``` bash
# vi modules/vpc/main.tf

resource "aws_vpc" "main" {
  cidr_block           = var.vpc_cidr
  enable_dns_support   = true
  enable_dns_hostnames = true

  tags = {
    Name = "${var.project}-vpc"
  }
}

resource "aws_subnet" "public" {
  count                   = length(var.public_subnets)
  vpc_id                  = aws_vpc.main.id
  cidr_block              = element(var.public_subnets, count.index)
  map_public_ip_on_launch = true
  availability_zone       = element(var.azs, count.index)

  tags = {
    Name = "${var.project}-public-${count.index + 1}"
  }
}

resource "aws_subnet" "private" {
  count             = length(var.private_subnets)
  vpc_id            = aws_vpc.main.id
  cidr_block        = element(var.private_subnets, count.index)
  availability_zone = element(var.azs, count.index)

  tags = {
    Name = "${var.project}-private-${count.index + 1}"
  }
}

resource "aws_internet_gateway" "igw" {
  vpc_id = aws_vpc.main.id

  tags = {
    Name = "${var.project}-igw"
  }
}

resource "aws_route_table" "public" {
  vpc_id = aws_vpc.main.id

  route {
    cidr_block = "0.0.0.0/0"
    gateway_id = aws_internet_gateway.igw.id
  }

  tags = {
    Name = "${var.project}-public-rt"
  }
}

resource "aws_route_table_association" "public" {
  count          = length(var.public_subnets)
  subnet_id      = element(aws_subnet.public.*.id, count.index)
  route_table_id = aws_route_table.public.id
}
```

``` bash
# modules/vpc/variables.tf

variable "project" {
  type        = string
  description = "Project name prefix"
}

variable "vpc_cidr" {
  type        = string
  description = "CIDR block for the VPC"
}

variable "public_subnets" {
  type        = list(string)
  description = "List of CIDRs for public subnets"
}

variable "private_subnets" {
  type        = list(string)
  description = "List of CIDRs for private subnets"
}

variable "azs" {
  type        = list(string)
  description = "Availability zones"
}
```

``` bash
# modules/vpc/outputs.tf

output "vpc_id" {
  value = aws_vpc.main.id
}

output "public_subnet_ids" {
  value = aws_subnet.public[*].id
}

output "private_subnet_ids" {
  value = aws_subnet.private[*].id
}
```


# 2. Public and Private Subnets with NAT Gateways

- `Objectives`: Implement a secure network architecture with public and private subnets. Use a NAT Gataway for private subnet internet access.

- `Steps`:

1. Set up a public subnets for resources accessible from the internets.
2. Create a private subnets for resources with no direct internet access.
3. Configure a NAT Gateway for private subnet internet access.

- **Terraform Configuration:**

Then I also define the terraform configuration for subnets. Since I have VPC module so I just adjust the `main.tf` file to suit the NAT configuration.

``` bash
cat >> /modules/vpc/main.tf

# Elastic IP for NAT Gateway
resource "aws_eip" "nat" {
  vpc = true

  tags = {
    Name = "${var.project}-nat-eip"
  }
}

# NAT Gateway in the first public subnet
resource "aws_nat_gateway" "nat" {
  allocation_id = aws_eip.nat.id
  subnet_id     = aws_subnet.public[0].id

  tags = {
    Name = "${var.project}-nat-gateway"
  }

  depends_on = [aws_internet_gateway.igw]
}

# Private Route Table
resource "aws_route_table" "private" {
  vpc_id = aws_vpc.main.id

  route {
    cidr_block     = "0.0.0.0/0"
    nat_gateway_id = aws_nat_gateway.nat.id
  }

  tags = {
    Name = "${var.project}-private-rt"
  }
}

# Associate Private Subnets with Private Route Table
resource "aws_route_table_association" "private" {
  count          = length(var.private_subnets)
  subnet_id      = aws_subnet.private[count.index].id
  route_table_id = aws_route_table.private.id
}
```



# 3. AWS MySQL RDS Setup

- **Objective:** Deploy a managed MySQL databash using Amazon RDS for WordPress data storage.

**Steps:**

1. Create an Amazon RDS instance with the MySQL engine.
2. Configure security groups for the RDS instance.
3. Connect WordPress to the RDS database.

- **Terraform Configuration:**

Then I created another directory for RDS which is `modules/rds`


``` bash
mkdir -p modules/rds

cd modules/rds
```

Then provide all the neccessary files needed in the same directory.


``` bash
# modules/rds/main.tf

resource "aws_db_subnet_group" "this" {
  name       = "${var.project}-rds-subnet-group"
  subnet_ids = var.private_subnet_ids

  tags = {
    Name = "${var.project}-rds-subnet-group"
  }
}

resource "aws_db_instance" "this" {
  identifier         = "${var.project}-mysql-db"
  engine             = "mysql"
  engine_version     = var.engine_version
  instance_class     = var.instance_class
  allocated_storage  = var.allocated_storage
  storage_type       = "gp2"
  username           = var.db_username
  password           = var.db_password
  db_subnet_group_name = aws_db_subnet_group.this.name
  vpc_security_group_ids = var.db_security_group_ids
  publicly_accessible   = false
  skip_final_snapshot   = true
  multi_az              = false
  deletion_protection   = false

  tags = {
    Name = "${var.project}-mysql-db"
  }
}
```

``` bash
# modules/rds/variables.tf

variable "project" {
  description = "Project name"
  type        = string
}

variable "private_subnet_ids" {
  description = "List of private subnet IDs for RDS"
  type        = list(string)
}

variable "db_username" {
  description = "Master DB username"
  type        = string
}

variable "db_password" {
  description = "Master DB password"
  type        = string
  sensitive   = true
}

variable "db_security_group_ids" {
  description = "Security group IDs for RDS"
  type        = list(string)
}

variable "allocated_storage" {
  description = "DB allocated storage in GB"
  type        = number
  default     = 20
}

variable "instance_class" {
  description = "DB instance type"
  type        = string
  default     = "db.t3.micro"
}

variable "engine_version" {
  description = "MySQL engine version"
  type        = string
  default     = "8.0"
}
```

``` bash
# modules/outputs.tf

output "db_endpoint" {
  value = aws_db_instance.this.endpoint
}

output "db_identifier" {
  value = aws_db_instance.this.id
}
```

# 4. EFS Setup for WorkPress Files


- **Objective:** Utilize Amazon Elastic File System (EFS) to store WordPress files for scalable and shared access.

**Steps:**

- Create an EFS file system
- Mount the EFS file system on WordPress instances.
- Configure WordPress to use the shared file system.


**Terraform Comfiguration:**

Then I also created another directory for `modules/efs.`

``` bash
# vi modules/efs/main.tf

# EFS File System
resource "aws_efs_file_system" "this" {
  creation_token = "${var.project}-efs"
  encrypted      = true

  tags = {
    Name = "${var.project}-efs"
  }
}

# EFS Mount Targets in each private subnet
resource "aws_efs_mount_target" "this" {
  count          = length(var.private_subnet_ids)
  file_system_id = aws_efs_file_system.this.id
  subnet_id      = element(var.private_subnet_ids, count.index)
  security_groups = [var.efs_security_group_id]
}
```

``` bash
# vi modules/efs/variables.tf

variable "project" {
  description = "Project name"
  type        = string
}

variable "private_subnet_ids" {
  description = "Private subnet IDs for mounting EFS"
  type        = list(string)
}

variable "efs_security_group_id" {
  description = "Security group ID for EFS"
  type        = string
}
```


# 5. Application Loas Balancer

- **Objective:** Set up an application Load Balancer to distribute incoming traffic among multiple instances, ensuring high availability and fault tolerance.

**Steps:**

- Create an application Loas Balancer
- Configure listener rules for routing traffic to instances.
- Integrate Load Balancer with Auto Scaling group.

**Terraform Configuration:**

- Uses Terraform to define Application Load Balancer configurations.
- Integrate Load Balancer with Auto Scaling group.

I also created another directory for `ALB` which is `modules/alb`.

``` bash
mkdir -p modules/alb

cd modules/alb
```

Then I created the neccessary files needed like main.tf, variables.tf and outputs.tf.

``` bash
# vi modules/alb/main.tf

# ALB Security Group (optional if not defined elsewhere)
resource "aws_security_group" "alb_sg" {
  name        = "${var.project}-alb-sg"
  description = "Allow HTTP inbound traffic"
  vpc_id      = var.vpc_id

  ingress {
    from_port   = 80
    to_port     = 80
    protocol    = "tcp"
    cidr_blocks = ["0.0.0.0/0"]
  }

  egress {
    from_port   = 0
    to_port     = 0
    protocol    = "-1"
    cidr_blocks = ["0.0.0.0/0"]
  }

  tags = {
    Name = "${var.project}-alb-sg"
  }
}

# ALB
resource "aws_lb" "this" {
  name               = "${var.project}-alb"
  internal           = false
  load_balancer_type = "application"
  security_groups    = [aws_security_group.alb_sg.id]
  subnets            = var.public_subnet_ids

  tags = {
    Name = "${var.project}-alb"
  }
}

# Target Group
resource "aws_lb_target_group" "this" {
  name     = "${var.project}-tg"
  port     = 80
  protocol = "HTTP"
  vpc_id   = var.vpc_id

  health_check {
    path                = "/"
    interval            = 30
    timeout             = 5
    healthy_threshold   = 2
    unhealthy_threshold = 2
    matcher             = "200"
  }

  tags = {
    Name = "${var.project}-tg"
  }
}

# Listener
resource "aws_lb_listener" "http" {
  load_balancer_arn = aws_lb.this.arn
  port              = 80
  protocol          = "HTTP"

  default_action {
    type             = "forward"
    target_group_arn = aws_lb_target_group.this.arn
  }
}
```

``` bash
# vi modules/alb/variables.tf

variable "project" {
  type        = string
  description = "Project name"
}

variable "vpc_id" {
  type        = string
  description = "VPC ID"
}

variable "public_subnet_ids" {
  type        = list(string)
  description = "List of public subnet IDs for ALB"
}
```

``` bash
# vi modules/alb/outputs.tf

output "alb_dns_name" {
  value = aws_lb.this.dns_name
}

output "alb_sg_id" {
  value = aws_security_group.alb_sg.id
}

output "target_group_arn" {
  value = aws_lb_target_group.this.arn
}
```


# 6. Auto Scaling Group

- **Objective:** Implement Auto Scaling to automatically adjust the number of instances based on traffic load.

**Steps:**

- Create an auto Scaling group.
- Define scaling policies based on metirics like CPU utilization.
- Configure launch configuration for instances.


**Terraform Configuration:**


- Develop Terraform scripts for Auto Scaling group creation.
- Define scaling policies and launch configurations.

Then I also created anothetr directory which is `modules/asg`

``` bash
mkdir -p molules/asg
cd molules/asg
```

Then I provided the neccessary scripts files for Auto Scaling Group.

``` bash
# vi molules/asg/main.tf

# Launch Template
resource "aws_launch_template" "this" {
  name_prefix   = "${var.project}-lt"
  image_id      = var.ami_id
  instance_type = var.instance_type
  key_name      = var.key_name

  network_interfaces {
    associate_public_ip_address = true
    security_groups             = var.ec2_security_group_ids
  }

  user_data = base64encode(var.user_data)

  tag_specifications {
    resource_type = "instance"

    tags = {
      Name = "${var.project}-wordpress"
    }
  }
}

# Auto Scaling Group
resource "aws_autoscaling_group" "this" {
  name                      = "${var.project}-asg"
  min_size                  = var.min_size
  max_size                  = var.max_size
  desired_capacity          = var.desired_capacity
  vpc_zone_identifier       = var.public_subnet_ids
  target_group_arns         = [var.alb_target_group_arn]
  health_check_type         = "EC2"
  health_check_grace_period = 300
  launch_template {
    id      = aws_launch_template.this.id
    version = "$Latest"
  }
  tag {
    key                 = "Name"
    value               = "${var.project}-wordpress"
    propagate_at_launch = true
  }

  lifecycle {
    create_before_destroy = true
  }
}

# Scaling policy: scale out
resource "aws_autoscaling_policy" "scale_out" {
  name                   = "${var.project}-scale-out"
  scaling_adjustment     = 1
  adjustment_type        = "ChangeInCapacity"
  cooldown               = 300
  autoscaling_group_name = aws_autoscaling_group.this.name
}

# CloudWatch alarm to trigger scale out
resource "aws_cloudwatch_metric_alarm" "cpu_high" {
  alarm_name          = "${var.project}-cpu-high"
  comparison_operator = "GreaterThanThreshold"
  evaluation_periods  = 2
  metric_name         = "CPUUtilization"
  namespace           = "AWS/EC2"
  period              = 60
  statistic           = "Average"
  threshold           = 70
  alarm_description   = "Trigger scaling out when CPU > 70%"
  dimensions = {
    AutoScalingGroupName = aws_autoscaling_group.this.name
  }
  alarm_actions = [aws_autoscaling_policy.scale_out.arn]
}
```

``` bash
# vi modules/asg/variables.tf

variable "project" {
  type = string
}

variable "ami_id" {
  type = string
}

variable "instance_type" {
  type = string
  default = "t3.micro"
}

variable "key_name" {
  type = string
}

variable "ec2_security_group_ids" {
  type = list(string)
}

variable "public_subnet_ids" {
  type = list(string)
}

variable "alb_target_group_arn" {
  type = string
}

variable "min_size" {
  type = number
  default = 1
}

variable "max_size" {
  type = number
  default = 3
}

variable "desired_capacity" {
  type = number
  default = 1
}

variable "user_data" {
  type = string
}
```

``` bash
# modules/asg/outputs.tf

output "asg_name" {
  value = aws_autoscaling_group.this.name
}
```

# 7. Root Directory

Then I moved back to the root directory and created main.tf, user_data.sh and terraform.tfvars

``` bash

# vi main.tf

provider "aws" {
  region = var.aws_region
}

module "vpc" {
  source           = "./modules/vpc"
  project          = var.project
  vpc_cidr         = var.vpc_cidr
  public_subnets   = var.public_subnets
  private_subnets  = var.private_subnets
  azs              = var.azs
}

module "security_group" {
  source  = "./modules/security-group"
  vpc_id  = module.vpc.vpc_id
}

module "rds" {
  source                = "./modules/rds"
  project               = var.project
  private_subnet_ids    = module.vpc.private_subnet_ids
  db_username           = var.db_username
  db_password           = var.db_password
  db_security_group_ids = [module.security_group.rds_sg_id]
}

module "efs" {
  source                = "./modules/efs"
  project               = var.project
  private_subnet_ids    = module.vpc.private_subnet_ids
  efs_security_group_id = module.security_group.efs_sg_id
}

module "alb" {
  source             = "./modules/alb"
  project            = var.project
  vpc_id             = module.vpc.vpc_id
  public_subnet_ids  = module.vpc.public_subnet_ids
}

module "asg" {
  source                 = "./modules/asg"
  project                = var.project
  ami_id                 = var.ami_id
  instance_type          = var.instance_type
  key_name               = var.key_name
  ec2_security_group_ids = [module.security_group.ec2_sg_id]
  public_subnet_ids      = module.vpc.public_subnet_ids
  alb_target_group_arn   = module.alb.target_group_arn
  user_data              = file("${path.module}/scripts/user_data.sh")
}
```

``` bash
# terraform.tfvars

project         = "digitalboost"
aws_region      = "us-east-1"
vpc_cidr        = "10.0.0.0/16"
public_subnets  = ["10.0.1.0/24", "10.0.2.0/24"]
private_subnets = ["10.0.3.0/24", "10.0.4.0/24"]
azs             = ["us-east-1a", "us-east-1b"]

db_username     = "admin"
db_password     = "StrongPassword123!"

ami_id          = "ami-0c02fb55956c7d316"  # Amazon Linux 2
instance_type   = "t3.micro"
key_name        = "your-keypair-name"
```

- I created a script folder in the root directory and cd into it.


``` bash
mkdir -p script

cd script
```

``` bash
mkdir scripts

vi scripts/user_data.sh

#!/bin/bash
yum update -y
yum install -y httpd php php-mysqlnd amazon-efs-utils

# Enable Apache
systemctl start httpd
systemctl enable httpd

# Mount EFS
mkdir -p /var/www/html
mount -t efs -o tls ${efs_dns_name}:/ /var/www/html

# Download and configure WordPress
cd /var/www/html
wget https://wordpress.org/latest.tar.gz
tar -xzf latest.tar.gz
cp -r wordpress/* .
rm -rf wordpress latest.tar.gz

# Configure wp-config.php
cp wp-config-sample.php wp-config.php
sed -i "s/database_name_here/${db_name}/" wp-config.php
sed -i "s/username_here/${db_username}/" wp-config.php
sed -i "s/password_here/${db_password}/" wp-config.php
sed -i "s/localhost/${db_endpoint}/" wp-config.php

chown -R apache:apache /var/www/html
```

![](./Images/1.%20tree.png)

- then I added permission to the `user_data.sh` execuatable permission.

``` bash
chmod +x scripts/data_data.sh
```

![](./Images/2.%20permision.png)


# 8. Terraform Initialization

1. After I provided all the files needed the I ran the following command.


``` bash
terraform init

terraform apply
```

# Result From the Terraform

- VPC
![](./Images/3.%20vpc.png)

- Subnets
![](./Images/3.%20Subnet%20Created.png)

- Gateway
![](./Images/4.%20Attached%20Successful.png)

- Database
![](./Images/5.%20DB%20Created.png)

- Wordpress page
![](./Images/6.%20WordPress%20Page.png)

![](./Images/7.%20WordPress.png)

- ALB working
![](./Images/8.%20ALB%20Working.png)

- Autoscaling 
![](./Images/9.%20AutoScaling%20Working.png)












