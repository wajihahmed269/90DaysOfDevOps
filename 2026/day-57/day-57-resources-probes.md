# Day 57 – Resource Requests, Limits, and Probes

## Resource Requests vs Limits

### Requests
Requests are the guaranteed minimum resources a Pod needs.
The Kubernetes scheduler uses requests to decide which node to place
a Pod on. If no node has enough available resources to satisfy the
request, the Pod stays Pending.

### Limits
Limits are the maximum resources a container is allowed to use.
The kubelet enforces limits at runtime — not at scheduling time.

### Key difference:
- Requests = promise to the scheduler (used for placement)
- Limits = hard ceiling enforced at runtime

### CPU vs Memory behavior when limits are exceeded:
- CPU is compressible — if a container exceeds its CPU limit,
  it gets throttled (slowed down) but keeps running
- Memory is incompressible — if a container exceeds its memory limit,
  it gets killed immediately with OOMKilled (Out Of Memory)

### CPU units:
- 1 CPU = 1000m (millicores)
- 100m = 0.1 CPU
- 500m = 0.5 CPU

### Memory units:
- Mi = mebibytes (1Mi = 1,048,576 bytes)
- Gi = gibibytes

---

## QoS Classes

Kubernetes assigns a Quality of Service class to every Pod
based on how requests and limits are configured.

| QoS Class | Condition | Priority |
|---|---|---|
| Guaranteed | requests == limits for all containers | Highest |
| Burstable | requests < limits (at least one container) | Medium |
| BestEffort | no requests or limits set at all | Lowest (evicted first) |

When the node runs out of memory, BestEffort pods are evicted first,
then Burstable, and Guaranteed pods are protected the longest.

---

## Task 1 – Resource Requests and Limits

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: resource-pod
spec:
  containers:
  - name: nginx
    image: nginx:1.25
    resources:
      requests:
        cpu: 100m
        memory: 128Mi
      limits:
        cpu: 250m
        memory: 256Mi
```

```bash
kubectl apply -f resource-pod.yaml
kubectl describe pod resource-pod
```

Inspected the Requests, Limits, and QoS Class sections.
Since requests and limits were different, QoS class was Burstable.

---

## Task 2 – OOMKilled

Created a Pod using the stress image with a memory limit of 100Mi
but instructed it to allocate 200M of memory:

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: oom-pod
spec:
  containers:
  - name: stress
    image: polinux/stress
    resources:
      limits:
        memory: 100Mi
    command: ["stress"]
    args: ["--vm", "1", "--vm-bytes", "200M", "--vm-hang", "1"]
```

The container was killed immediately.

```bash
kubectl describe pod oom-pod
```

Output showed:
- Reason: OOMKilled
- Exit Code: 137 (128 + SIGKILL)

Key learning: CPU over limit = throttled and still running.
Memory over limit = killed instantly with no warning.
Exit code 137 always means OOMKilled.

---

## Task 3 – Pending Pod (Requesting Too Much)

Created a Pod requesting cpu: 100 cores and memory: 128Gi —
far more than any node has available.

```bash
kubectl describe pod <pending-pod-name>
```

Events section showed the scheduler message:
0/1 nodes are available: insufficient cpu, insufficient memory.

The Pod stayed Pending forever — it was never scheduled because
no node could satisfy the resource request.

Key learning: Always set realistic requests. Oversized requests
cause Pending pods even when the cluster has capacity.

---

## Task 4 – Liveness Probe

A liveness probe detects stuck or broken containers.
If the probe fails consecutively, Kubernetes restarts the container.

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: liveness-pod
spec:
  containers:
  - name: busybox
    image: busybox
    command: ["sh", "-c", "touch /tmp/healthy && sleep 30 && rm /tmp/healthy && sleep 3600"]
    livenessProbe:
      exec:
        command: ["cat", "/tmp/healthy"]
      initialDelaySeconds: 5
      periodSeconds: 5
      failureThreshold: 3
```

What happened:
- Container started and created /tmp/healthy
- Liveness probe checked the file every 5 seconds — passed
- After 30 seconds the file was deleted
- 3 consecutive probe failures triggered a container restart
- kubectl get pod -w showed RESTARTS count increasing

Key learning: Liveness probe failure = container restart.
Use it to recover from deadlocks or stuck processes.

---

## Task 5 – Readiness Probe

A readiness probe controls whether a Pod receives traffic.
Failure removes the Pod from Service endpoints but does NOT restart it.

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: readiness-pod
spec:
  containers:
  - name: nginx
    image: nginx:1.25
    readinessProbe:
      httpGet:
        path: /
        port: 80
      initialDelaySeconds: 5
      periodSeconds: 5
```

```bash
kubectl expose pod readiness-pod --port=80 --name=readiness-svc
kubectl get endpoints readiness-svc
```

Pod IP was listed in endpoints — traffic was being sent to it.

Broke the probe by deleting the index file:
```bash
kubectl exec readiness-pod -- rm /usr/share/nginx/html/index.html
```

After 15 seconds:
- Pod showed 0/1 READY
- kubectl get endpoints readiness-svc showed empty endpoints
- Container was NOT restarted — just removed from traffic rotation

Key learning: Readiness failure = no traffic. Liveness failure = restart.
These are different tools for different problems.

---

## Task 6 – Startup Probe

A startup probe gives slow-starting containers extra time to initialize.
While the startup probe is running, liveness and readiness probes are
completely disabled. This prevents premature restarts during startup.

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: startup-pod
spec:
  containers:
  - name: busybox
    image: busybox
    command: ["sh", "-c", "sleep 20 && touch /tmp/started && sleep 3600"]
    startupProbe:
      exec:
        command: ["cat", "/tmp/started"]
      periodSeconds: 5
      failureThreshold: 12
    livenessProbe:
      exec:
        command: ["cat", "/tmp/started"]
      periodSeconds: 10
```

failureThreshold: 12 with periodSeconds: 5 gives a 60 second budget
for the container to start. The liveness probe only kicks in after
the startup probe succeeds.

If failureThreshold were 2 instead of 12, the startup probe would
fail after just 10 seconds and Kubernetes would kill the container
before it had a chance to finish starting — causing an infinite
restart loop.

---

## Probe Types

| Type | How it checks | Use case |
|---|---|---|
| exec | Runs a command inside the container | File existence, custom scripts |
| httpGet | Makes an HTTP GET request | Web servers, REST APIs |
| tcpSocket | Opens a TCP connection | Databases, non-HTTP services |

## Probe Parameters

| Parameter | Meaning |
|---|---|
| initialDelaySeconds | Wait before first probe |
| periodSeconds | How often to probe |
| failureThreshold | Consecutive failures before action |
| successThreshold | Consecutive successes to recover |
| timeoutSeconds | How long to wait for a response |

---

## Liveness vs Readiness vs Startup Summary

| Probe | Failure action | Purpose |
|---|---|---|
| Liveness | Restart container | Recover from deadlocks and crashes |
| Readiness | Remove from endpoints | Stop traffic during temporary issues |
| Startup | Kill container | Give slow apps time to initialize |

---

## Task 7 – Cleanup

```bash
kubectl delete pod resource-pod oom-pod liveness-pod readiness-pod startup-pod
kubectl delete service readiness-svc
```

Verified with kubectl get pods and kubectl get services.

---

## Key Takeaways

- Always set requests and limits on production workloads
- Requests are for scheduling, limits are for runtime enforcement
- CPU over limit = throttled, memory over limit = OOMKilled
- Exit code 137 always means OOMKilled
- QoS Guaranteed = requests equal limits, highest protection
- QoS BestEffort = no requests or limits, evicted first
- Liveness probe restarts stuck containers
- Readiness probe removes unhealthy pods from traffic
- Startup probe protects slow-starting containers from being killed
- Never set requests higher than your node capacity
