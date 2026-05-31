Day 66 – Provision an EKS Cluster with Terraform Modules
Overview

Day 66 focused on provisioning a complete Kubernetes cluster on AWS using Terraform.

A production-style EKS cluster with managed node groups and networking was deployed fully through Infrastructure as Code.

Architecture

Terraform
|
AWS VPC Module
|
EKS Module
|
Managed Node Group
|
Kubernetes Cluster
|
Nginx Deployment + LoadBalancer Service

Concepts Used
Concept	Used For
EKS	Managed Kubernetes on AWS
VPC Module	Kubernetes networking
Managed Node Groups	Worker nodes
kubectl	Cluster interaction
Kubernetes Deployment	Running applications
LoadBalancer Service	External access
Key Learnings
Provisioned EKS using Terraform modules
Created private and public subnets
Connected kubectl to the EKS cluster
Deployed Nginx workloads
Learned EKS networking and managed node groups
Destroyed the cluster cleanly to avoid AWS costs
