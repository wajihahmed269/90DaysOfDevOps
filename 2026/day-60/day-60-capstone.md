   Day 60 – Capstone: WordPress + MySQL on Kubernetes

## Overview

This capstone brings together every major Kubernetes concept learned
across Days 52 to 59 into one real production-style deployment.
A full WordPress + MySQL stack running on Kubernetes with namespacing,
persistent storage, secrets management, self-healing, and autoscaling.

---

## Architecture
Browser
|
NodePort Service (port 30080)
|
WordPress Deployment (2 replicas)
|  envFrom: ConfigMap (DB host, DB name)
|  secretKeyRef: Secret (DB user, DB password)
|  livenessProbe + readinessProbe on /wp-login.php
|
Headless Service (mysql)
|
MySQL StatefulSet (mysql-0)
|  envFrom: Secret (root password, database, user, password)
|  volumeClaimTemplate: 1Gi PVC at /var/lib/mysql
|
PersistentVolume (1Gi, dynamic provisioning)

---

## Concepts Used

| Concept | Day Learned | Used For |
|---|---|---|
| Namespace | Day 52 | Isolated capstone environment |
| Deployment | Day 52 | WordPress with 2 replicas |
| StatefulSet | Day 56 | MySQL with stable identity |
| Headless Service | Day 56 | Stable DNS for mysql-0 |
| NodePort Service | Day 53 | External access to WordPress |
| ConfigMap | Day 54 | WordPress DB host and name |
| Secret | Day 54 | MySQL and WordPress credentials |
| PVC via volumeClaimTemplates | Day 55 | MySQL persistent data |
| Resource requests and limits | Day 57 | Both WordPress and MySQL |
| Liveness probe | Day 57 | Restart broken WordPress pods |
| Readiness probe | Day 57 | Traffic control for WordPress |
| HPA | Day 58 | Auto-scale WordPress under load |

---

## Task 1 – Namespace

Created and switched to the capstone namespace:

```bash
kubectl create namespace capstone
kubectl config set-context --current --namespace=capstone
```

All resources were created inside this namespace.
Deleting the namespace at the end removes everything at once.

---

## Task 2 – MySQL Deployment

### Secret

```yaml
apiVersion: v1
kind: Secret
metadata:
  name: mysql-secret
  namespace: capstone
stringData:
  MYSQL_ROOT_PASSWORD: rootpassword123
  MYSQL_DATABASE: wordpress
  MYSQL_USER: wpuser
  MYSQL_PASSWORD: wppassword123
```

Used stringData so values do not need to be manually base64 encoded.
Kubernetes handles the encoding automatically.

### Headless Service

```yaml
apiVersion: v1
kind: Service
metadata:
  name: mysql
  namespace: capstone
spec:
  clusterIP: None
  selector:
    app: mysql
  ports:
  - port: 3306
    targetPort: 3306
```

clusterIP: None makes this a Headless Service.
Gives mysql-0 its own stable DNS name:
mysql-0.mysql.capstone.svc.cluster.local

### StatefulSet

```yaml
apiVersion: apps/v1
kind: StatefulSet
metadata:
  name: mysql
  namespace: capstone
spec:
  serviceName: mysql
  replicas: 1
  selector:
    matchLabels:
      app: mysql
  template:
    metadata:
      labels:
        app: mysql
    spec:
      containers:
      - name: mysql
        image: mysql:8.0
        envFrom:
        - secretRef:
            name: mysql-secret
        ports:
        - containerPort: 3306
        resources:
          requests:
            cpu: 250m
            memory: 512Mi
          limits:
            cpu: 500m
            memory: 1Gi
        volumeMounts:
        - name: mysql-data
          mountPath: /var/lib/mysql
  volumeClaimTemplates:
  - metadata:
      name: mysql-data
    spec:
      accessModes:
      - ReadWriteOnce
      resources:
        requests:
          storage: 1Gi
```

Verified MySQL was working:
```bash
kubectl exec -it mysql-0 -- mysql -u wpuser -pwppassword123 -e "SHOW DATABASES;"
```

Output confirmed the wordpress database existed.

---

## Task 3 – WordPress Deployment

### ConfigMap

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: wordpress-config
  namespace: capstone
data:
  WORDPRESS_DB_HOST: mysql-0.mysql.capstone.svc.cluster.local:3306
  WORDPRESS_DB_NAME: wordpress
```

The DB host uses the full StatefulSet DNS pattern so WordPress
always connects to the stable mysql-0 pod regardless of restarts.

### Deployment

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: wordpress
  namespace: capstone
spec:
  replicas: 2
  selector:
    matchLabels:
      app: wordpress
  template:
    metadata:
      labels:
        app: wordpress
    spec:
      containers:
      - name: wordpress
        image: wordpress:latest
        ports:
        - containerPort: 80
        envFrom:
        - configMapRef:
            name: wordpress-config
        env:
        - name: WORDPRESS_DB_USER
          valueFrom:
            secretKeyRef:
              name: mysql-secret
              key: MYSQL_USER
        - name: WORDPRESS_DB_PASSWORD
          valueFrom:
            secretKeyRef:
              name: mysql-secret
              key: MYSQL_PASSWORD
        resources:
          requests:
            cpu: 200m
            memory: 256Mi
          limits:
            cpu: 500m
            memory: 512Mi
        livenessProbe:
          httpGet:
            path: /wp-login.php
            port: 80
          initialDelaySeconds: 60
          periodSeconds: 15
          failureThreshold: 5
        readinessProbe:
          httpGet:
            path: /wp-login.php
            port: 80
          initialDelaySeconds: 30
          periodSeconds: 10
          failureThreshold: 5
```

Both pods came up 1/1 Running after about 2 minutes.

Non-sensitive config (DB host, DB name) came from the ConfigMap.
Sensitive credentials (DB user, DB password) came from the Secret.
This separation follows the principle of least exposure.

---

## Task 4 – Expose WordPress

```yaml
apiVersion: v1
kind: Service
metadata:
  name: wordpress
  namespace: capstone
spec:
  type: NodePort
  selector:
    app: wordpress
  ports:
  - port: 80
    targetPort: 80
    nodePort: 30080
```

```bash
minikube service wordpress -n capstone
```

WordPress setup wizard was accessible in the browser.
Completed setup and created a test blog post.

---

## Task 5 – Self-Healing and Persistence Tests

### WordPress self-healing test:
Deleted one WordPress pod — the Deployment recreated it within seconds.
Refreshed the browser — site was still fully accessible.

### MySQL self-healing test:
Deleted mysql-0 — the StatefulSet recreated it with the same name
and automatically reconnected to the same PVC.

### Data persistence test:
After both pods were deleted and recreated, the test blog post
was still visible in WordPress.

Data survived because it was stored in the PersistentVolume,
not inside the container filesystem. This proved the PVC worked correctly.

### Key difference observed:
- WordPress pod (Deployment) got a new random name after deletion
- MySQL pod (StatefulSet) came back as mysql-0 with the same identity
- WordPress reconnected to MySQL using the stable DNS name, not an IP

---

## Task 6 – Horizontal Pod Autoscaler

```yaml
apiVersion: autoscaling/v2
kind: HorizontalPodAutoscaler
metadata:
  name: wordpress-hpa
  namespace: capstone
spec:
  scaleTargetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: wordpress
  minReplicas: 2
  maxReplicas: 10
  metrics:
  - type: Resource
    resource:
      name: cpu
      target:
        type: Utilization
        averageUtilization: 50
```

HPA configured to maintain minimum 2 replicas and scale up to 10
when average CPU exceeds 50% of requests.

```bash
kubectl get hpa -n capstone
kubectl get all -n capstone
```

Full stack was running — StatefulSet, Deployment, three Services,
HPA, and PVC all working together.

---

## Task 7 – Bonus Helm Comparison

Installed WordPress using Bitnami Helm chart:
```bash
helm install wp-helm bitnami/wordpress --namespace helm-test --create-namespace
```

Helm created significantly more resources automatically including
ingress, service accounts, and additional configmaps that would
require manual work in the raw YAML approach.

Manual approach gives more control and understanding of each resource.
Helm approach is faster and includes production best practices by default.

For learning: manual approach.
For production: Helm chart from a trusted source.

Cleaned up:
```bash
helm uninstall wp-helm -n helm-test
kubectl delete namespace helm-test
```

---

## Task 8 – Cleanup

Took final screenshot of kubectl get all -n capstone showing
all resources running together.

Deleted everything:
```bash
kubectl delete namespace capstone
kubectl config set-context --current --namespace=default
```

Deleting the namespace removed every resource inside it automatically
— Deployment, StatefulSet, Services, ConfigMap, Secret, PVC, HPA
all gone in one command.

---

## Reflection

### What was hardest:
Getting the WORDPRESS_DB_HOST exactly right using the full
StatefulSet DNS pattern. A single typo here caused WordPress
to fail to connect to MySQL silently.

### What clicked:
How all the pieces connect — Secret feeds into StatefulSet and
Deployment, ConfigMap provides non-sensitive config, Headless
Service enables stable DNS, PVC keeps data alive across restarts.
It all makes sense as a complete system now.

### What to add for production:
- Ingress controller with TLS instead of NodePort
- External Secrets Operator for secrets from AWS Secrets Manager
- Network Policies to restrict pod-to-pod communication
- Pod Disruption Budgets to ensure availability during node maintenance
- Multiple MySQL replicas with read/write splitting
- Backup solution for the MySQL PVC
- Monitoring with Prometheus and Grafana

---
