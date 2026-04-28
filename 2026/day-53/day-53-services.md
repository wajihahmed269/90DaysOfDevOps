# Day 53 – Kubernetes Services

## What Problem Do Services Solve?

Every Pod in Kubernetes gets its own IP address. But there are two big problems:
1. Pod IPs are not stable — when a Pod restarts it gets a brand new IP
2. A Deployment runs multiple Pods — you cant connect to all of them individually

A Service solves both problems by providing:
- A stable IP and DNS name that never changes even if pods restart
- Automatic load balancing across all pods that match its selector
- ---

## Task 1 – Deployed the Application

Created a Deployment with 3 replicas of nginx:

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: web-app
  labels:
    app: web-app
spec:
  replicas: 3
  selector:
    matchLabels:
      app: web-app
  template:
    metadata:
      labels:
        app: web-app
    spec:
      containers:
      - name: nginx
        image: nginx:1.25
        ports:
        - containerPort: 80
```

```bash
kubectl apply -f app-deployment.yaml
kubectl get pods -o wide
```

All 3 pods came up running with individual IPs:
- web-app-6cffb4b956-2ghzd — 10.244.0.4
- web-app-6cffb4b956-j5dn6 — 10.244.0.5
- web-app-6cffb4b956-jpl9q — 10.244.0.6

These IPs are random and will change if pods restart.
This is exactly the problem Services fix.

---

## Task 2 – ClusterIP Service (Internal Access Only)

ClusterIP is the default Service type. It gives pods a stable internal IP
that is only reachable from inside the cluster. Outside traffic cannot reach it.

```yaml
apiVersion: v1
kind: Service
metadata:
  name: web-app-clusterip
spec:
  type: ClusterIP
  selector:
    app: web-app
  ports:
  - port: 80
    targetPort: 80
```

### Field explanations:
- `selector: app: web-app` — routes traffic to all pods with this label
- `port: 80` — the port the Service listens on
- `targetPort: 80` — the port on the actual pod to forward traffic to

```bash
kubectl apply -f clusterip-service.yaml
kubectl get services
```

Service created with CLUSTER-IP: 10.97.199.41
This IP is stable and will not change even if all pods restart.

### Tested from inside the cluster:
```bash
kubectl run test-client --image=busybox:latest --rm -it --restart=Never -- sh
wget -qO- http://web-app-clusterip
```

Got the nginx welcome page successfully. The service load balanced
the request to one of the 3 running pods automatically.

---

## Task 3 – Kubernetes DNS

Kubernetes has a built-in DNS server. Every Service automatically gets
a DNS name in this format:

Tested both the short name and full DNS name from inside a pod:

```bash
kubectl run dns-test --image=busybox:latest --rm -it --restart=Never -- sh

# Short name
wget -qO- http://web-app-clusterip

# Full DNS name
wget -qO- http://web-app-clusterip.default.svc.cluster.local

# DNS lookup
nslookup web-app-clusterip
```

Both worked and returned the nginx welcome page.
nslookup confirmed the DNS resolved to 10.97.199.41 which matches
the CLUSTER-IP from kubectl get services.

Use the short name within the same namespace.
Use the full DNS name when reaching across namespaces.

---

## Task 4 – NodePort Service (External Access)

NodePort opens a port on the node itself so traffic can reach the
service from outside the cluster.

```yaml
apiVersion: v1
kind: Service
metadata:
  name: web-app-nodeport
spec:
  type: NodePort
  selector:
    app: web-app
  ports:
  - port: 80
    targetPort: 80
    nodePort: 30080
```

- `nodePort: 30080` — port opened on the node (must be 30000-32767)
- Traffic flow: NodeIP:30080 -> Service -> Pod:80

```bash
kubectl apply -f nodeport-service.yaml
minikube service web-app-nodeport --url
```

Got URL: http://192.168.49.2:30080
Accessed nginx welcome page from outside the cluster successfully.

---

## Task 5 – LoadBalancer Service (Cloud Access)

In cloud environments like AWS, a LoadBalancer Service provisions a
real external load balancer automatically. On Minikube the EXTERNAL-IP
shows pending because there is no real cloud provider — this is expected.

```yaml
apiVersion: v1
kind: Service
metadata:
  name: web-app-loadbalancer
spec:
  type: LoadBalancer
  selector:
    app: web-app
  ports:
  - port: 80
    targetPort: 80
```

```bash
kubectl apply -f loadbalancer-service.yaml
kubectl get services
```

EXTERNAL-IP showed pending on Minikube as expected.
On real AWS EKS this would show a public IP or hostname automatically.

---

## Task 6 – Service Types Compared

```bash
kubectl get services -o wide
kubectl describe service web-app-loadbalancer
```

| Type | Accessible From | Use Case |
|---|---|---|
| ClusterIP | Inside cluster only | Internal microservice communication |
| NodePort | Outside via NodeIP:Port | Development and testing |
| LoadBalancer | Outside via cloud load balancer | Production on AWS/GCP/Azure |

Key insight: Each type builds on the previous one.
LoadBalancer also has a NodePort and ClusterIP inside it.
Confirmed this with kubectl describe — all three were visible
in the LoadBalancer service description.

---

## Task 7 – Cleanup

```bash
kubectl delete -f app-deployment.yaml
kubectl delete -f clusterip-service.yaml
kubectl delete -f nodeport-service.yaml
kubectl delete -f loadbalancer-service.yaml

kubectl get pods
kubectl get services
```

All resources deleted. Only the default kubernetes service remained.

---

## Key Takeaways

- Services give Pods a stable IP and DNS name that never changes
- ClusterIP is for internal communication between services
- NodePort exposes the app on a port on the node for external access
- LoadBalancer provisions a cloud load balancer for production traffic
- Kubernetes DNS automatically creates a name for every service
- The selector in a Service must match the labels on the Pods exactly
- LoadBalancer builds on NodePort which builds on ClusterIP
- Use kubectl get endpoints to see which Pod IPs a service is routing to
- 
