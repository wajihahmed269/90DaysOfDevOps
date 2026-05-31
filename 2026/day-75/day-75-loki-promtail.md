Day 75 – Log Management with Loki and Promtail
Overview

Day 75 introduced centralized log management using Loki and Promtail.

This completed the second pillar of observability: logs.

Architecture

Docker Containers
|
Promtail
|
Loki
|
Grafana
|
User

Concepts Used
Concept	Used For
Loki	Log aggregation
Promtail	Log shipping
LogQL	Querying logs
Docker Logs	Container log collection
Grafana Explore	Log analysis
Key Learnings
Configured Loki for centralized logs
Shipped Docker logs with Promtail
Queried logs using LogQL
Connected Loki with Grafana
Practiced troubleshooting using logs
