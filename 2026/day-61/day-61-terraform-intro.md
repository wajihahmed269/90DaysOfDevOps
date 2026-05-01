# Day 61 – Introduction to Terraform and Your First AWS Infrastructure

## What is Infrastructure as Code (IaC)?

Infrastructure as Code means defining your servers, networks, databases,
and cloud resources in code files instead of clicking through a UI.
You can version it, review it, repeat it, and share it like any other code.
If something breaks, you delete it and recreate it identically in minutes.

## What problems does IaC solve?

- No more "it works on my AWS account" — everyone gets identical infrastructure
- No more forgetting what you clicked — the code is the documentation
- No more manual mistakes — same code runs the same way every time
- Easy to destroy and recreate entire environments on demand
- Changes go through code review just like application code

## How is Terraform different from other tools?

| Tool | Made by | Works with |
|---|---|---|
| Terraform | HashiCorp | Any cloud — AWS, GCP, Azure, and more |
| CloudFormation | AWS | AWS only |
| Ansible | Red Hat | Server configuration, not infrastructure |
| Pulumi | Pulumi | Any cloud but uses real programming languages |

Terraform is declarative — you describe WHAT you want, not HOW to build it.
You write "I want an EC2 instance" and Terraform figures out all the API
calls needed to make it happen. It is also cloud-agnostic — the same
tool and workflow works across every major cloud provider.

---

## Task 1 – Install Terraform

HashiCorp does not yet support Ubuntu Resolute in their apt repository
so Terraform was installed manually:

```bash
wget https://releases.hashicorp.com/terraform/1.7.5/terraform_1.7.5_linux_amd64.zip
sudo apt install unzip -y
unzip terraform_1.7.5_linux_amd64.zip
sudo mv terraform /usr/local/bin/
terraform -version
```

Verified Terraform v1.7.5 was installed successfully.

---

## Task 2 – Configure AWS CLI

Installed AWS CLI and configured credentials:

```bash
sudo apt install awscli -y
aws configure
```

Entered:
- AWS Access Key ID — from IAM Security credentials
- AWS Secret Access Key — from IAM Security credentials
- Default region — eu-north-1
- Output format — json

Verified AWS connection:
```bash
aws sts get-caller-identity
```

Output showed Account ID and ARN confirming successful connection to AWS.

---

## Task 3 – First Terraform Config: S3 Bucket

Created a project directory:
```bash
mkdir terraform-basics && cd terraform-basics
```

Created main.tf:
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
  region = "eu-north-1"
}

resource "aws_s3_bucket" "my_bucket" {
  bucket = "terraweek-ahmed-2026"

  tags = {
    Name = "TerraWeek Bucket"
  }
}
```

### Ran the Terraform lifecycle:

```bash
terraform init
```
Downloaded the AWS provider plugin into the .terraform/ directory.
The .terraform/ directory contains provider binaries that Terraform
uses to communicate with AWS APIs.

```bash
terraform plan
```
Showed a preview of exactly what would be created — no changes made yet.
Always read the plan before applying.

```bash
terraform apply
```
Typed yes to confirm. S3 bucket was created in AWS.

Verified the bucket existed in the AWS S3 console.

---

## Task 4 – Add an EC2 Instance

Found the correct AMI ID for eu-north-1 region:
```bash
aws ec2 describe-images --owners amazon \
  --filters "Name=name,Values=amzn2-ami-hvm-*-x86_64-gp2" \
  --query 'Images[0].ImageId' --output text --region eu-north-1
```

Added EC2 instance block to main.tf:
```hcl
resource "aws_instance" "my_ec2" {
  ami           = "ami-015869825c7c8ac64"
  instance_type = "t3.micro"

  tags = {
    Name = "TerraWeek-Day1"
  }
}
```

Note: eu-north-1 uses t3.micro for free tier, not t2.micro.
t2.micro is only free tier eligible in older regions like us-east-1.

```bash
terraform plan
terraform apply
```

Terraform detected the S3 bucket already existed in state and only
created the EC2 instance — Plan showed 1 to add, 0 to change.

Verified the instance was running in the AWS EC2 console with
the tag Name = TerraWeek-Day1.

### How Terraform knew the S3 bucket already existed:
Terraform stores everything it creates in terraform.tfstate.
When you run plan or apply, Terraform reads the state file,
compares it to your .tf files, and only acts on the differences.
The S3 bucket was already in state so Terraform skipped it.

---

## Task 5 – The State File

Inspected the state file:
```bash
cat terraform.tfstate
terraform show
terraform state list
terraform state show aws_s3_bucket.my_bucket
terraform state show aws_instance.my_ec2
```

### What the state file stores:
- Resource type and name
- All attributes of the resource (ID, ARN, region, tags, etc.)
- Dependencies between resources---

## Task 6 – Modify, Plan, and Destroy

Changed the EC2 tag from TerraWeek-Day1 to TerraWeek-Modified in main.tf.

```bash
terraform plan
```

Plan output symbols:
- ~ means in-place update — resource stays, only attribute changes
- + means creating a new resource
- - means destroying a resource
- -/+ means destroy and recreate (happens with AMI or instance type changes)

The tag change showed ~ (in-place update) — no rebuild needed.

```bash
terraform apply
```

Verified the tag changed in the AWS EC2 console.

Destroyed everything:
```bash
terraform destroy
```

Typed yes to confirm. Both the S3 bucket and EC2 instance were deleted.
Verified in AWS console — both resources were gone.

---

## Terraform Commands Summary

| Command | What it does |
|---|---|
| terraform init | Downloads provider plugins, sets up the project |
| terraform plan | Shows what will be created, changed, or destroyed |
| terraform apply | Executes the plan and creates/modifies resources |
| terraform destroy | Destroys all resources managed by Terraform |
| terraform show | Human readable view of current state |
| terraform state list | Lists all resources in state |
| terraform fmt | Auto-formats .tf files |
| terraform validate | Checks for syntax errors |

---

## Key Takeaways

- Terraform lets you create cloud infrastructure using code files
- Declarative means you describe the desired state, not the steps
- terraform init must be run first in every new project
- Always run terraform plan before apply to review changes
- The state file is Terraform's memory — never edit it manually
- Never commit state files or access keys to Git
- AMI IDs and free tier instance types vary by region
- eu-north-1 uses t3.micro for free tier, not t2.micro
- terraform destroy removes everything cleanly in one command
- IaC makes infrastructure reproducible, reviewable, and version controlled
- Provider metadata

### Why never manually edit the state file:
Terraform uses the state file as its source of truth. If you edit it
manually and make a mistake, Terraform will think resources exist that
do not, or vice versa — causing it to try to destroy or recreate things
incorrectly. Always use terraform state commands to interact with state.

### Why not commit state to Git:
The state file can contain sensitive information like passwords, keys,
and resource IDs. It should be stored in a remote backend like S3
with DynamoDB locking for team environments.

Added to .gitignore:
