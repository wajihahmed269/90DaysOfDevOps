# Day 58 – Metrics Server and Horizontal Pod Autoscaler (HPA)

## What is the Metrics Server?

The Metrics Server is a cluster-wide aggregator of resource usage data.
It collects CPU and memory usage from every node and pod by polling
the kubelet on each node every 15 seconds.

Without the Metrics Server:
- kubectl top nodes and kubectl top pods do not work
- HPA cannot see actual usage and cannot make scaling decisions

The Metrics Server is not installed by default on most clusters.
On Minikube it is available as an addon.

---

## What is HPA?

A Horizontal Pod Autoscaler automatically scales the number of pod
replicas in a Deployment, StatefulSet, or ReplicaSet based on
observed resource usage.

When traffic increases, CPU goes up, HPA adds replicas.
When traffic drops, CPU goes down, HPA removes replicas.

This is how production systems handle variable traffic without
wasting money on idle capacity.

### HPA formula:
Example: 2 replicas at 80% CPU with 50% target = ceil(2 * 80/50) = 4 replicas

---

## Task 1 – Install Metrics Server

Checked if already running:
```bash
kubectl get pods -n kube-system | grep metrics-server
```

Enabled it on Minikube:
```bash
minikube addons enable metrics-server
```

Waited 60 seconds then verified:
```bash
kubectl top nodes
kubectl top pods -A
```

Output showed real-time CPU and memory usage for the node and
all running pods. Metrics Server was working correctly.

---

## Task 2 – Explore kubectl top

```bash
kubectl top nodes
kubectl top pods -A
kubectl top pods -A --sort-by=cpu
```

Key learning: kubectl top shows ACTUAL real-time usage.
kubectl describe pod shows CONFIGURED requests and limits.
These are completely different things.

- Actual usage can be lower than requests (pod is underutilizing)
- Actual usage can never exceed limits (kubelet enforces them)
- HPA uses actual usage from Metrics Server to make decisions

Data from Metrics Server is refreshed every 15 seconds so
kubectl top values update approximately every 15 seconds.

---

## Task 3 – Deployment with CPU Requests

Created a Deployment using the hpa-example image:

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: php-apache
spec:
  replicas: 1
  selector:
    matchLabels:
      app: php-apache
  template:
    metadata:
      labels:
        app: php-apache
    spec:
      containers:
      - name: php-apache
        image: registry.k8s.io/hpa-example
        ports:
        - containerPort: 80
        resources:
          requests:
            cpu: 200m
```

CPU request of 200m is mandatory for HPA to work.
HPA calculates utilization as a percentage of the request.
Without requests, HPA shows unknown in the TARGETS column
and cannot make any scaling decisions.

Exposed as a Service:
```bash
kubectl expose deployment php-apache --port=80
```

---

## Task 4 – Create HPA (Imperative)

```bash
kubectl autoscale deployment php-apache --cpu-percent=50 --min=1 --max=10
```

This means:
- Scale up when average CPU exceeds 50% of requests (50% of 200m = 100m)
- Minimum 1 replica always running
- Maximum 10 replicas allowed

Checked status:
```bash
kubectl get hpa
kubectl describe hpa php-apache
```

TARGETS showed current CPU percentage vs the 50% target.
Initially showed unknown for about 30 seconds while metrics arrived.

---

## Task 5 – Generate Load and Watch Autoscaling

Started a load generator pod:
```bash
kubectl run load-generator --image=busybox:1.36 --restart=Never -- \
  /bin/sh -c "while true; do wget -q -O- http://php-apache; done"
```

Watched HPA respond:
```bash
kubectl get hpa php-apache --watch
```

What happened over 1-3 minutes:
- CPU climbed above 50% of requests
- HPA calculated desired replicas using the formula
- Replicas increased automatically to distribute the load
- CPU per pod stabilized back near the 50% target

Stopped the load:
```bash
kubectl delete pod load-generator
```

Scale-down is intentionally slow — HPA has a 5 minute stabilization
window before scaling down to avoid flapping. This is by design.

---

## Task 6 – HPA from YAML (Declarative)

Deleted the imperative HPA:
```bash
kubectl delete hpa php-apache
```

Created HPA using autoscaling/v2 API:

```yaml
apiVersion: autoscaling/v2
kind: HorizontalPodAutoscaler
metadata:
  name: php-apache
spec:
  scaleTargetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: php-apache
  minReplicas: 1
  maxReplicas: 10
  metrics:
  - type: Resource
    resource:
      name: cpu
      target:
        type: Utilization
        averageUtilization: 50
  behavior:
    scaleUp:
      stabilizationWindowSeconds: 0
      policies:
      - type: Pods
        value: 4
        periodSeconds: 15
    scaleDown:
      stabilizationWindowSeconds: 300
      policies:
      - type: Pods
        value: 1
        periodSeconds: 60
```

### What the behavior section controls:
- scaleUp stabilizationWindowSeconds: 0 — scale up immediately with no delay
- scaleUp policy: add up to 4 pods every 15 seconds
- scaleDown stabilizationWindowSeconds: 300 — wait 5 minutes before scaling down
- scaleDown policy: remove only 1 pod every 60 seconds (gradual scale down)

This prevents thrashing — rapid scale up and down in response to
short spikes in traffic.

### autoscaling/v1 vs autoscaling/v2:

| Feature | v1 | v2 |
|---|---|---|
| CPU scaling | Yes | Yes |
| Memory scaling | No | Yes |
| Custom metrics | No | Yes |
| Behavior control | No | Yes |
| Multiple metrics | No | Yes |

Always use autoscaling/v2 for production — v1 is too limited.

---

## Task 7 – Cleanup

```bash
kubectl delete hpa php-apache
kubectl delete service php-apache
kubectl delete deployment php-apache
kubectl delete pod load-generator
```

Verified with kubectl get pods and kubectl get hpa.
Metrics Server was left installed for future use.

---

## How HPA Scaling Works Step by Step

1. Load generator sends traffic to php-apache Service
2. CPU usage on the pod increases above 50% of requests
3. Metrics Server collects the usage data every 15 seconds
4. HPA controller reads the metrics every 15 seconds
5. HPA calculates desired replicas using the formula
6. HPA updates the Deployment replica count
7. Deployment creates new pods to handle the load
8. CPU per pod drops back toward the 50% target
9. Load stops — CPU drops below target
10. After 5 minute stabilization window HPA scales back down

---

## Key Takeaways

- Metrics Server collects real-time CPU and memory from all nodes
- kubectl top shows actual usage, not configured requests or limits
- HPA requires resources.requests to calculate utilization percentages
- Without requests, TARGETS shows unknown and HPA cannot scale
- HPA checks metrics every 15 seconds
- Scale-up is fast, scale-down has a 5 minute stabilization window
- autoscaling/v2 supports memory, custom metrics, and behavior control
- The behavior section prevents flapping during traffic spikes
- HPA works with Deployments, StatefulSets, and ReplicaSets
- Always test HPA with a load generator before going to production
