Day 63 – Variables, Outputs, Data Sources and Expressions
Overview

Day 63 focused on transforming Terraform code from hardcoded infrastructure into reusable and environment-aware configurations.

Variables, outputs, data sources, locals, and expressions were used to make the infrastructure dynamic and scalable.

Concepts Used
Concept	Used For
Variables	Dynamic configuration
Outputs	Returning infrastructure values
tfvars	Environment-specific configuration
Data Sources	Fetching existing AWS information
Locals	Reusable expressions
Conditional Expressions	Environment-aware logic
Terraform Functions	String, collection, and networking operations
Key Learnings
Removed hardcoded AMI IDs and instance types
Used terraform.tfvars and prod.tfvars
Used outputs to expose EC2 IPs and IDs
Used data sources for dynamic AMI lookup
Used locals and merge functions for standardized tagging
Practiced Terraform functions using terraform console
