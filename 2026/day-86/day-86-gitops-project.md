Day 86 – GitOps Project: End-to-End CI/CD Pipeline
Overview

Day 86 completed the GitOps workflow by connecting GitHub Actions with ArgoCD and Kubernetes.

A full automated deployment pipeline was created from code push to production deployment.

Architecture

Developer Push
|
GitHub Actions
|
Docker Build + Push
|
Manifest Update
|
Git Commit
|
ArgoCD Sync
|
EKS Deployment

Concepts Used
Concept	Used For
GitHub Actions	CI automation
DockerHub	Image registry
GitOps Workflow	Deployment automation
Manifest Updates	Image versioning
Rolling Updates	Zero downtime deployments
Drift Recovery	Cluster consistency
Key Learnings
Built complete GitOps CI/CD automation
Automated Docker image builds and pushes
Updated Kubernetes manifests automatically
Triggered ArgoCD deployments from Git changes
Practiced zero-downtime deployment workflows
Tested drift detection and recovery
