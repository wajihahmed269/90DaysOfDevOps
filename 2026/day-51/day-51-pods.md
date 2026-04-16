Day 51 – Kubernetes Manifests and Your First Pods
The Four Required Fields

apiVersion — which version of the Kubernetes API to use. Pods use v1. Other resources use different groups like apps/v1
kind — the resource type you're creating: Pod, Deployment, Service, etc.
metadata — the identity of the resource. name is required. labels are optional key-value pairs used to select and filter resources later
spec — the desired state. For a Pod, this means what containers to run, which images, ports, resource limits, etc.

Task 1: nginx-pod.yaml
yamlapiVersion: v1
kind: Pod
metadata:
  name: nginx-pod
  labels:
    app: nginx
spec:
  containers:
  - name: nginx
    image: nginx:latest
    ports:
    - containerPort: 80
bashkubectl apply -f nginx-pod.yaml
kubectl get pods -o wide
kubectl exec -it nginx-pod -- /bin/bash
# inside: curl localhost:80
Task 2: busybox-pod.yaml
yamlapiVersion: v1
kind: Pod
metadata:
  name: busybox-pod
  labels:
    app: busybox
    environment: dev
spec:
  containers:
  - name: busybox
    image: busybox:latest
    command: ["sh", "-c", "echo Hello from BusyBox && sleep 3600"]
bashkubectl apply -f busybox-pod.yaml
kubectl logs busybox-pod
# Hello from BusyBox
Without sleep 3600, BusyBox runs the echo, exits immediately, and Kubernetes keeps restarting it — that's a CrashLoopBackOff. The sleep keeps the container alive.
Task 3: Imperative vs Declarative
Imperative — you tell Kubernetes exactly what action to take right now:
bashkubectl run redis-pod --image=redis:latest
Quick, no YAML needed, but not repeatable, not reviewable, not stored in git.
Declarative — you describe the desired state in a file and apply it:
bashkubectl apply -f nginx-pod.yaml
Repeatable, version-controllable, and idempotent — running it again changes nothing if the state already matches.
The dry-run trick:
bashkubectl run test-pod --image=nginx --dry-run=client -o yaml > test-pod.yaml
Generates a valid manifest without creating anything — best way to scaffold a starting point.
Task 4: Validation
bashkubectl apply -f nginx-pod.yaml --dry-run=client
kubectl apply -f nginx-pod.yaml --dry-run=server
When image field is removed, the error is:
error: error validating: spec.containers[0].image: Required value
--dry-run=client catches YAML structure errors locally. --dry-run=server sends it to the API server which also catches semantic errors like invalid image formats or 
missing required fields.
Task 5: Third Pod with Labels
yamlapiVersion: v1
kind: Pod
metadata:
  name: app-pod
  labels:
    app: webapp
    environment: staging
    team: backend
spec:
  containers:
  - name: nginx
    image: nginx:alpine
    ports:
    - containerPort: 80
bashkubectl get pods --show-labels
kubectl get pods -l environment=staging
kubectl get pods -l team=backend
kubectl label pod nginx-pod environment=production
kubectl label pod nginx-pod environment-   # removes the label
Task 6: What Happens When You Delete a Standalone Pod
It's gone permanently. No controller is watching it, so nothing recreates it. This is why bare Pods are only for learning and debugging. In production every pod is managed
by a Deployment which has a ReplicaSet watching it — if a pod dies, the ReplicaSet immediately creates a replacement. Day 52 is exactly that.
