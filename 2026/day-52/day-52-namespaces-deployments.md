# Day 52 – Kubernetes Namespaces and Deployments

## What are Namespaces?
Namespaces are a way to organize and isolate resources inside a Kubernetes cluster.
Think of them like folders — different teams or environments (dev, staging, production)
get their own folder so they don't interfere with each other.

Kubernetes comes with 4 built-in namespaces:
- `default` — where resources go if you don't specify a namespace
- `kube-system` — Kubernetes internal components (API server, scheduler, etc.)
- `kube-public` — publicly readable resources
- `kube-node-lease` — tracks node heartbeats to check if nodes are alive

## Why Use Namespaces?
- Separate environments (dev vs staging vs production)
- Avoid accidental changes to wrong resources
- Apply different permissions per namespace
- Organize large clusters with many teams

---

## Task 1 – Explored Default Namespaces

Listed all built-in namespaces:
```bash
kubectl get namespaces
kubectl get pods -n kube-system
```
Observed all control plane components running inside kube-system.
These are the internal pods keeping the cluster alive — we do not touch them.

---

## Task 2 – Created Custom Namespaces

Created dev and staging using imperative commands:
```bash
kubectl create namespace dev
kubectl create namespace staging
```

Created production namespace using a manifest file:
```yaml
apiVersion: v1
kind: Namespace
metadata:
  name: production
```
```bash
kubectl apply -f namespace.yaml
```

Ran pods inside specific namespaces:
```bash
kubectl run nginx-dev --image=nginx:latest -n dev
kubectl run nginx-staging --image=nginx:latest -n staging
```

Key learning: `kubectl get pods` only shows the default namespace.
You must use `-n <namespace>` or `-A` to see pods in other namespaces.

---

## Task 3 – Created First Deployment

A Deployment tells Kubernetes to keep X replicas of a Pod running at all times.
If a Pod crashes, the Deployment controller automatically recreates it.

### nginx-deployment.yaml
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-deployment
  namespace: dev
  labels:
    app: nginx
spec:
  replicas: 3
  selector:
    matchLabels:
      app: nginx
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
      - name: nginx
        image: nginx:1.24
        ports:
        - containerPort: 80
```

### Explanation of each section:
- `apiVersion: apps/v1` — Deployments belong to the apps API group
- `replicas: 3` — Kubernetes will always maintain exactly 3 pods
- `selector.matchLabels` — connects the Deployment to the pods it manages
- `template` — the blueprint used to create each pod
- `image: nginx:1.24` — specific version pinned for rollback purposes

Applied the deployment:
```bash
kubectl apply -f nginx-deployment.yaml
kubectl get deployments -n dev
kubectl get pods -n dev
```

Output showed 3 pods running with names like `nginx-deployment-xxxxx-yyyyy`.

### Column meanings:
- READY — how many pods are running out of desired (e.g. 3/3)
- UP-TO-DATE — pods updated to the latest configuration
- AVAILABLE — pods ready to serve traffic

---

## Task 4 – Self-Healing

Deleted one pod manually to test self-healing:
```bash
kubectl delete pod <pod-name> -n dev
kubectl get pods -n dev
```

The Deployment controller detected only 2 of 3 desired replicas were running
and immediately created a brand new replacement pod within seconds.
The new pod had a different name — it is a completely new pod, not the same one restarted.

Key difference vs standalone Pod:
- Standalone Pod — deleted = gone forever, nothing recreates it
- Deployment Pod — deleted = Kubernetes automatically creates a replacement

---

## Task 5 – Scaling

### Imperative (quick command):
```bash
# Scale up
kubectl scale deployment nginx-deployment --replicas=5 -n dev

# Scale down
kubectl scale deployment nginx-deployment --replicas=2 -n dev
```

When scaled down from 5 to 2, the extra 3 pods went into Terminating
state and were removed. Kubernetes always matches the desired replica count.

### Declarative (proper DevOps way):
Changed `replicas: 4` in nginx-deployment.yaml and ran:
```bash
kubectl apply -f nginx-deployment.yaml
```
This is the preferred approach — the YAML file becomes the source of truth.

---

## Task 6 – Rolling Update and Rollback

### Rolling Update
Updated the nginx image version:
```bash
kubectl set image deployment/nginx-deployment nginx=nginx:1.25 -n dev
kubectl rollout status deployment/nginx-deployment -n dev
```

Kubernetes replaced pods one by one — a new pod comes up healthy first,
then the old one is terminated. This gives zero downtime during updates.

### Rollback
Checked rollout history:
```bash
kubectl rollout history deployment/nginx-deployment -n dev
```

Rolled back to previous version:
```bash
kubectl rollout undo deployment/nginx-deployment -n dev
```

Verified rollback:
```bash
kubectl describe deployment nginx-deployment -n dev | grep Image
```
Image returned to nginx:1.24 after rollback.

---

## Task 7 – Cleanup

```bash
kubectl delete deployment nginx-deployment -n dev
kubectl delete pod nginx-dev -n dev
kubectl delete pod nginx-staging -n staging
kubectl delete namespace dev staging production
```

Deleting a namespace removes everything inside it automatically.
Verified with `kubectl get namespaces` and `kubectl get pods -A` — only default namespaces remained.

---

## Key Takeaways
- Namespaces isolate resources — always use them in real projects
- Deployments are the right way to run apps in Kubernetes, not standalone Pods
- Self-healing means Kubernetes automatically replaces crashed pods
- Rolling updates give zero downtime deployments
- Rollbacks let you recover instantly if something breaks
- Always pin image versions (nginx:1.24 not nginx:latest) so rollbacks work properly
