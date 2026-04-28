# Day 54 – Kubernetes ConfigMaps and Secrets

## What are ConfigMaps and Secrets?

### ConfigMaps
ConfigMaps store non-sensitive configuration data as key-value pairs.
Instead of hardcoding config inside your container image, you store it
in a ConfigMap and inject it into your pods. This means you can change
config without rebuilding your image.

Use ConfigMaps for:
- Environment names (production, staging, dev)
- Feature flags
- App settings like ports and debug modes
- Full config files like nginx.conf

### Secrets
Secrets store sensitive data like passwords, API keys, and tokens.
They work the same way as ConfigMaps but are treated with more care
by Kubernetes — stored in tmpfs (memory) on nodes, can be encrypted
at rest, and access can be restricted with RBAC.

Use Secrets for:
- Database passwords
- API keys
- TLS certificates
- Any sensitive credentials

---

## Task 1 – ConfigMap from Literals

Created a ConfigMap directly from the command line:

```bash
kubectl create configmap app-config \
  --from-literal=APP_ENV=production \
  --from-literal=APP_DEBUG=false \
  --from-literal=APP_PORT=8080
```

Inspected it:
```bash
kubectl describe configmap app-config
kubectl get configmap app-config -o yaml
```

All three key-value pairs were visible in plain text:
- APP_ENV: production
- APP_DEBUG: false
- APP_PORT: 8080

Key learning: ConfigMap data is stored as plain text with zero encoding.
Never store passwords or sensitive data in a ConfigMap.

---

## Task 2 – ConfigMap from a File

Created a custom nginx config file with a /health endpoint:

```nginx
server {
    listen 80;
    location /health {
        return 200 'healthy';
        add_header Content-Type text/plain;
    }
}
```

Created ConfigMap from the file:
```bash
kubectl create configmap nginx-config --from-file=default.conf=nginx-health.conf
```

The key name `default.conf` becomes the actual filename when the
ConfigMap is mounted as a volume inside a pod.

Verified with:
```bash
kubectl get configmap nginx-config -o yaml
```

The entire file contents were stored under the key `default.conf`.

---

## Task 3 – Using ConfigMaps in Pods

### Method 1 – Environment Variables with envFrom

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: app-env-pod
spec:
  containers:
  - name: busybox
    image: busybox
    command: ["sh", "-c", "echo APP_ENV=$APP_ENV && echo APP_PORT=$APP_PORT && echo APP_DEBUG=$APP_DEBUG && sleep 3600"]
    envFrom:
    - configMapRef:
        name: app-config
  restartPolicy: Never
```

`envFrom` injects ALL keys from the ConfigMap as environment variables
at once. No need to list them individually.

Verified with:
```bash
kubectl logs app-env-pod
```

Output showed all three values correctly injected into the container.

### Method 2 – Volume Mount for Config Files

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: nginx-config-pod
spec:
  containers:
  - name: nginx
    image: nginx:1.25
    volumeMounts:
    - name: nginx-config-vol
      mountPath: /etc/nginx/conf.d
  volumes:
  - name: nginx-config-vol
    configMap:
      name: nginx-config
  restartPolicy: Never
```

The ConfigMap was mounted as a file at /etc/nginx/conf.d/default.conf
inside the container.

Tested the health endpoint:
```bash
kubectl exec nginx-config-pod -- curl -s http://localhost/health
```

Returned: healthy

### When to use which method:
- Environment variables — simple key-value settings
- Volume mounts — full config files like nginx.conf or app.properties

---

## Task 4 – Creating a Secret

Created a secret with database credentials:

```bash
kubectl create secret generic db-credentials \
  --from-literal=DB_USER=admin \
  --from-literal=DB_PASSWORD=s3cureP@ssw0rd
```

Inspected it:
```bash
kubectl get secret db-credentials -o yaml
```

The values appeared base64 encoded — for example DB_USER showed as `YWRtaW4=`

Decoded it:
```bash
echo 'YWRtaW4=' | base64 --decode
```

Output: admin

### Why base64 is NOT encryption:
Base64 is just encoding — it converts binary data to text format.
Anyone with cluster access can decode it instantly. It is not
protection. Real security comes from:
- RBAC — restricting who can read secrets
- Encryption at rest — encrypting secrets in etcd
- External secret managers like AWS Secrets Manager or HashiCorp Vault

---

## Task 5 – Using Secrets in a Pod

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: secret-pod
spec:
  containers:
  - name: busybox
    image: busybox
    command: ["sh", "-c", "echo DB_USER=$DB_USER && cat /etc/db-credentials/DB_PASSWORD && sleep 3600"]
    env:
    - name: DB_USER
      valueFrom:
        secretKeyRef:
          name: db-credentials
          key: DB_USER
    volumeMounts:
    - name: secret-vol
      mountPath: /etc/db-credentials
      readOnly: true
  volumes:
  - name: secret-vol
    secret:
      secretName: db-credentials
  restartPolicy: Never
```

Verified with:
```bash
kubectl logs secret-pod
```

Key learning: When secrets are mounted as volumes or injected as
environment variables, Kubernetes automatically decodes the base64.
The pod always sees the plain text value — not the encoded version.

Each Secret key becomes a separate file in the mounted directory.
The file content is the decoded plaintext value.

---

## Task 6 – ConfigMap Live Update

Created a ConfigMap with a message:
```bash
kubectl create configmap live-config --from-literal=message=hello
```

Created a pod that reads the mounted file every 5 seconds:

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: live-pod
spec:
  containers:
  - name: busybox
    image: busybox
    command: ["sh", "-c", "while true; do cat /etc/config/message; echo ''; sleep 5; done"]
    volumeMounts:
    - name: config-vol
      mountPath: /etc/config
  volumes:
  - name: config-vol
    configMap:
      name: live-config
  restartPolicy: Never
```

Updated the ConfigMap without touching the pod:
```bash
kubectl patch configmap live-config --type merge -p '{"data":{"message":"world"}}'
```

After 30-60 seconds the pod output automatically changed from
`hello` to `world` without any pod restart.

### Key difference — volume mounts vs environment variables:
- Volume mounted ConfigMaps — update automatically within 30-60 seconds
- Environment variables — never update, they are set at pod startup only
  and require a pod restart to pick up new values

---

## Task 7 – Cleanup

```bash
kubectl delete pod app-env-pod nginx-config-pod secret-pod live-pod
kubectl delete configmap app-config nginx-config live-config
kubectl delete secret db-credentials
```

Verified everything was removed with:
```bash
kubectl get pods
kubectl get configmaps
kubectl get secrets
```

Only the default kubernetes service account secret remained.

---

## Key Takeaways

- ConfigMaps store non-sensitive config — env settings, feature flags, config files
- Secrets store sensitive data — passwords, tokens, API keys
- envFrom injects all ConfigMap keys as environment variables at once
- Volume mounts are used for full config files
- base64 is encoding not encryption — never rely on it for security
- Volume mounted ConfigMaps update automatically without pod restart
- Environment variables only update on pod restart
- Always use RBAC to control who can read Secrets in production
- For real security use external secret managers like AWS Secrets Manager
