# Day 59 – Helm: Kubernetes Package Manager

## What is Helm?

Helm is the package manager for Kubernetes — like apt for Ubuntu
or pip for Python. Instead of writing and managing dozens of
individual YAML files for every application, Helm bundles everything
into a single package called a chart.

### Three core concepts:

- Chart — a package of Kubernetes manifest templates. Contains
  Deployments, Services, ConfigMaps, Secrets and more, all templated
  so values can be customized at install time.

- Release — a specific installation of a chart in your cluster.
  You can install the same chart multiple times with different names
  and different values — each is a separate release.

- Repository — a collection of charts hosted online, like a package
  registry. Bitnami, Artifact Hub, and official project repos all
  host charts.

---

## Task 1 – Install Helm

Installed Helm using the official install script:
```bash
curl https://raw.githubusercontent.com/helm/helm/main/scripts/get-helm-3 | bash
```

Verified:
```bash
helm version
helm env
```

helm version showed the installed Helm version.
helm env showed the local Helm environment configuration including
the cache and data directories.

---

## Task 2 – Add a Repository and Search

Added the Bitnami repository:
```bash
helm repo add bitnami https://charts.bitnami.com/bitnami
helm repo update
```

Searched for charts:
```bash
helm search repo nginx
helm search repo bitnami
```

helm search repo bitnami listed all available Bitnami charts.
Bitnami maintains a large collection of production-ready charts
for common applications like nginx, postgresql, redis, kafka, and more.

---

## Task 3 – Install a Chart

Deployed nginx with one command:
```bash
helm install my-nginx bitnami/nginx
```

Checked what was created:
```bash
kubectl get all
helm list
helm status my-nginx
helm get manifest my-nginx
```

One helm install command automatically created:
- A Deployment
- A Service
- A ConfigMap
- A ServiceAccount

This replaced writing and applying multiple YAML files by hand.
helm get manifest showed the exact Kubernetes manifests that
were generated and applied by Helm.

---

## Task 4 – Customize with Values

Viewed all available customization options:
```bash
helm show values bitnami/nginx
```

Installed a custom release using --set flags:
```bash
helm install nginx-custom bitnami/nginx --set replicaCount=3 --set service.type=NodePort
```

Created a custom-values.yaml file:
```yaml
replicaCount: 3

service:
  type: NodePort

resources:
  requests:
    cpu: 100m
    memory: 128Mi
  limits:
    cpu: 250m
    memory: 256Mi
```

Installed using the values file:
```bash
helm install nginx-values bitnami/nginx -f custom-values.yaml
```

Verified the overrides were applied:
```bash
helm get values nginx-values
```

### --set vs -f values.yaml:
- --set is good for quick single overrides on the command line
- -f values.yaml is better for multiple values and version control
- Both can be combined — --set overrides take precedence over the file

---

## Task 5 – Upgrade and Rollback

Upgraded the release to 5 replicas:
```bash
helm upgrade my-nginx bitnami/nginx --set replicaCount=5
```

Checked revision history:
```bash
helm history my-nginx
```

Showed revision 1 (original install) and revision 2 (upgrade).

Rolled back to revision 1:
```bash
helm rollback my-nginx 1
helm history my-nginx
```

After rollback, history showed 3 revisions:
- Revision 1 — original install
- Revision 2 — upgrade to 5 replicas
- Revision 3 — rollback to revision 1

Key learning: Rollback creates a NEW revision rather than deleting
the history. This gives a full audit trail of every change made
to the release. Same concept as Deployment rollouts from Day 52
but at the full application stack level.

---

## Task 6 – Create Your Own Chart

Scaffolded a new chart:
```bash
helm create my-app
```

### Chart directory structure:
