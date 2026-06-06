Day 82 – EKS Networking with Gateway API and Persistent Storage
Overview

Day 82 focused on production networking and storage inside Kubernetes using the Gateway API, Envoy Gateway, cert-manager, and AWS EBS storage.

This introduced next-generation Kubernetes traffic management beyond traditional Ingress resources.

Architecture

Internet
|
AWS Network Load Balancer
|
Envoy Gateway
|
Gateway API
|
HTTPRoute
|
BankApp Service
|
BankApp Pods

Concepts Used
Concept	Used For
Gateway API	Advanced Kubernetes traffic management
Envoy Gateway	Gateway controller
HTTPRoute	Traffic routing
TLS Termination	HTTPS traffic handling
cert-manager	Automated certificate management
Let's Encrypt	TLS certificate provider
BackendTrafficPolicy	Session persistence
EBS Volumes	Persistent storage
Key Learnings
Installed Envoy Gateway on EKS
Configured GatewayClass, Gateway, and HTTPRoute resources
Implemented TLS with cert-manager and Let's Encrypt
Configured cookie-based session affinity
Used EBS storage for MySQL and Ollama persistence
Learned modern Kubernetes networking architecture
