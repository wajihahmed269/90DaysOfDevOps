Day 81 – Introduction to Amazon EKS with Terraform
Overview

Day 81 focused on understanding Amazon EKS and provisioning a production-style Kubernetes cluster on AWS using Terraform.

This was the transition from local Kubernetes clusters like Kind into managed cloud-native infrastructure running on AWS.

The AI-BankApp Terraform configuration was studied in depth to understand networking, node groups, IAM integration, and EKS add-ons.

Architecture

Terraform
|
AWS VPC
|
Public + Private + Intra Subnets
|
Amazon EKS Control Plane
|
Managed Node Group
|
Kubernetes Worker Nodes
|
BankApp Pods

Concepts Used
Concept	Used For
Amazon EKS	Managed Kubernetes service
Managed Control Plane	AWS-managed Kubernetes core components
Node Groups	Worker nodes for pods
IAM Integration	Secure AWS access control
VPC Networking	Kubernetes networking
Private/Public Subnets	Isolated infrastructure layers
NAT Gateway	Outbound internet access
EKS Add-ons	Cluster networking and storage
Terraform Modules	Infrastructure provisioning
Key Learnings
Learned EKS architecture and managed Kubernetes concepts
Studied production Terraform configuration for EKS
Provisioned an EKS cluster using Terraform modules
Connected kubectl to the EKS cluster
Explored EKS add-ons including CoreDNS, VPC CNI, EBS CSI Driver, and Metrics Server
Verified worker nodes across multiple availability zones
