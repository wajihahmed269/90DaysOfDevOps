# Day 62 – Providers, Resources and Dependencies

## Overview

Day 62 focused on building a complete AWS networking stack using Terraform.

The goal was to understand how Terraform connects resources together, how providers work, and how Terraform decides the correct creation order through implicit and explicit dependencies.

This was the first step from writing standalone Terraform resources toward building real infrastructure.

---

## Architecture

User
|
Terraform AWS Provider
|
VPC
|
Public Subnet
|
Internet Gateway
|
Route Table
|
Route Table Association
|
Security Group
|
EC2 Instance
|
S3 Bucket for logs

---

## Concepts Used

| Concept                  | Used For                                              |
| ------------------------ | ----------------------------------------------------- |
| AWS Provider             | Connecting Terraform with AWS                         |
| Provider Version Pinning | Controlling which provider version Terraform installs |
| VPC                      | Main private network for infrastructure               |
| Subnet                   | Public subnet inside the VPC                          |
| Internet Gateway         | Internet access for the VPC                           |
| Route Table              | Routing public traffic to the internet                |
| Route Table Association  | Connecting subnet with route table                    |
| Security Group           | Controlling inbound and outbound traffic              |
| EC2 Instance             | Running a server inside the subnet                    |
| Implicit Dependency      | Terraform detects dependency from resource references |
| Explicit Dependency      | Manually defining order using `depends_on`            |
| Lifecycle Rule           | Controlling resource replacement behavior             |
| Terraform Graph          | Visualizing infrastructure dependencies               |

---

## Task 1 – AWS Provider Setup

Created a new Terraform project directory:

```bash
mkdir terraform-aws-infra
cd terraform-aws-infra
```

Created `providers.tf`:

```hcl
terraform {
  required_providers {
    aws = {
      source  = "hashicorp/aws"
      version = "~> 5.0"
    }
  }
}

provider "aws" {
  region = "ap-south-1"
}
```

Initialized Terraform:

```bash
terraform init
```

Terraform downloaded the AWS provider and created the provider lock file:

```bash
.terraform.lock.hcl
```

The lock file stores the exact provider version and checksums.
This makes sure the same provider version is used consistently across machines.

### Provider Version Meaning

| Version Constraint | Meaning                                                      |
| ------------------ | ------------------------------------------------------------ |
| `~> 5.0`           | Allows any compatible 5.x version, but not 6.x               |
| `>= 5.0`           | Allows version 5.0 or newer, including future major versions |
| `= 5.0.0`          | Allows only exactly version 5.0.0                            |

`~> 5.0` is safer than `>= 5.0` because it avoids unexpected breaking changes from major versions.

---

## Task 2 – VPC Infrastructure

Created a VPC:

```hcl
resource "aws_vpc" "main" {
  cidr_block = "10.0.0.0/16"

  tags = {
    Name = "TerraWeek-VPC"
  }
}
```

Created a public subnet inside the VPC:

```hcl
resource "aws_subnet" "public" {
  vpc_id                  = aws_vpc.main.id
  cidr_block              = "10.0.1.0/24"
  map_public_ip_on_launch = true

  tags = {
    Name = "TerraWeek-Public-Subnet"
  }
}
```

Created an internet gateway:

```hcl
resource "aws_internet_gateway" "igw" {
  vpc_id = aws_vpc.main.id

  tags = {
    Name = "TerraWeek-IGW"
  }
}
```

Created a route table:

```hcl
resource "aws_route_table" "public" {
  vpc_id = aws_vpc.main.id

  route {
    cidr_block = "0.0.0.0/0"
    gateway_id = aws_internet_gateway.igw.id
  }

  tags = {
    Name = "TerraWeek-Public-RT"
  }
}
```

Associated the route table with the public subnet:

```hcl
resource "aws_route_table_association" "public_assoc" {
  subnet_id      = aws_subnet.public.id
  route_table_id = aws_route_table.public.id
}
```

Ran:

```bash
terraform plan
terraform apply
```

Terraform created the networking resources in the correct order.

---

## Task 3 – Implicit Dependencies

Terraform automatically understands dependencies when one resource references another.

Example:

```hcl
vpc_id = aws_vpc.main.id
```

This tells Terraform that the subnet depends on the VPC.

### Implicit Dependencies Found

| Resource                | Depends On                | Reason                               |
| ----------------------- | ------------------------- | ------------------------------------ |
| Subnet                  | VPC                       | Uses `aws_vpc.main.id`               |
| Internet Gateway        | VPC                       | Uses `aws_vpc.main.id`               |
| Route Table             | VPC and Internet Gateway  | Uses VPC ID and gateway ID           |
| Route Table Association | Subnet and Route Table    | Uses subnet ID and route table ID    |
| EC2 Instance            | Subnet and Security Group | Uses subnet ID and security group ID |

Terraform uses these references to build a dependency graph.

If Terraform tried to create the subnet before the VPC, AWS would reject it because a subnet cannot exist without a VPC.

---

## Task 4 – Security Group and EC2 Instance

Created a security group:

```hcl
resource "aws_security_group" "web_sg" {
  name        = "terraweek-sg"
  description = "Allow SSH and HTTP"
  vpc_id      = aws_vpc.main.id

  ingress {
    description = "Allow SSH"
    from_port   = 22
    to_port     = 22
    protocol    = "tcp"
    cidr_blocks = ["0.0.0.0/0"]
  }

  ingress {
    description = "Allow HTTP"
    from_port   = 80
    to_port     = 80
    protocol    = "tcp"
    cidr_blocks = ["0.0.0.0/0"]
  }

  egress {
    description = "Allow all outbound traffic"
    from_port   = 0
    to_port     = 0
    protocol    = "-1"
    cidr_blocks = ["0.0.0.0/0"]
  }

  tags = {
    Name = "TerraWeek-SG"
  }
}
```

Created an EC2 instance:

```hcl
resource "aws_instance" "main" {
  ami                         = "ami-xxxxxxxxxxxxxxxxx"
  instance_type               = "t2.micro"
  subnet_id                   = aws_subnet.public.id
  vpc_security_group_ids      = [aws_security_group.web_sg.id]
  associate_public_ip_address = true

  tags = {
    Name = "TerraWeek-Server"
  }
}
```

The instance was created inside the public subnet and attached to the security group.

---

## Task 5 – Explicit Dependency with depends_on

Created an S3 bucket for application logs:

```hcl
resource "aws_s3_bucket" "logs" {
  bucket = "terraweek-app-logs-example"

  depends_on = [aws_instance.main]

  tags = {
    Name = "TerraWeek-Logs"
  }
}
```

Normally, this bucket has no direct dependency on the EC2 instance.

But with:

```hcl
depends_on = [aws_instance.main]
```

Terraform waits for the EC2 instance before creating the bucket.

### When to Use depends_on

Use `depends_on` when Terraform cannot automatically detect the dependency.

Examples:

| Scenario                                                     | Why depends_on Helps                           |
| ------------------------------------------------------------ | ---------------------------------------------- |
| App server must exist before log bucket setup                | No direct reference exists                     |
| IAM policy must exist before another service starts using it | Dependency may be logical, not visible in code |
| Kubernetes resources need a cluster ready first              | Provider may not infer all timing dependencies |

---

## Task 6 – Terraform Graph

Generated the dependency graph:

```bash
terraform graph
```

Or as an image:

```bash
terraform graph | dot -Tpng > graph.png
```

This graph shows how Terraform understands resource order.

The graph helps debug complex infrastructure, especially when many modules and resources depend on each other.

---

## Task 7 – Lifecycle Rule

Added lifecycle behavior to the EC2 instance:

```hcl
lifecycle {
  create_before_destroy = true
}
```

This tells Terraform to create the replacement instance before destroying the old one.

Useful when downtime should be reduced during replacement.

### Terraform Lifecycle Arguments

| Lifecycle Argument      | Purpose                                              |
| ----------------------- | ---------------------------------------------------- |
| `create_before_destroy` | Creates replacement resource before deleting old one |
| `prevent_destroy`       | Blocks accidental destruction of critical resources  |
| `ignore_changes`        | Ignores selected changes made outside Terraform      |

Example use cases:

| Argument                | Real Use                                      |
| ----------------------- | --------------------------------------------- |
| `create_before_destroy` | Replacing EC2 instances with less downtime    |
| `prevent_destroy`       | Protecting databases or production S3 buckets |
| `ignore_changes`        | Ignoring tags changed by external tools       |

---

## Task 8 – Destroy and Cleanup

Destroyed all resources:

```bash
terraform destroy
```

Terraform destroyed resources in reverse dependency order.

For example:

1. EC2 instance destroyed first
2. Security group destroyed after instance
3. Route table association removed
4. Route table removed
5. Internet gateway detached and deleted
6. Subnet deleted
7. VPC deleted

This prevents dependency errors during cleanup.

---

## Final Notes

Day 62 was important because it showed how Terraform thinks.

The main lesson was not just creating AWS resources.
The real lesson was understanding relationships between resources.

Terraform is powerful because it builds a dependency graph, creates infrastructure in the correct order, and destroys it safely in reverse order.

This is the foundation for writing production infrastructure as code.
