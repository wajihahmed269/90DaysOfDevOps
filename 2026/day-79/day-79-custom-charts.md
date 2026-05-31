Day 79 – Creating a Custom Helm Chart for AI-BankApp
Overview

Day 79 focused on converting the AI-BankApp Kubernetes manifests into a reusable Helm chart.

This reduced multiple raw YAML files into a configurable deployment package.

Concepts Used
Concept	Used For
Custom Helm Charts	Packaging applications
values.yaml	Configurable deployments
Templates	Dynamic Kubernetes manifests
ConfigMaps	Application configuration
Secrets	Credential management
PVC Templates	Persistent storage
HPA	Auto scaling
Key Learnings
Built a custom Helm chart from scratch
Converted raw manifests into templates
Used Helm functions like b64enc
Parameterized deployment configurations
Built reusable Kubernetes deployments
