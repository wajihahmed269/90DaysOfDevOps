# Day 55 – Persistent Volumes and Persistent Volume Claims

## Why Containers Need Persistent Storage

Containers are ephemeral by nature. When a Pod dies or gets deleted,
everything inside it disappears — including any data written to the
container filesystem. This is fine for stateless apps like web servers
but is a serious problem for databases and anything that needs to
survive restarts.
PV status lifecycle:
- Available — storage exists, nobody is using it
- Bound — matched to a PVC and in use
- Released — PVC was deleted, data still exists
- Failed — something went wrong

---

## Task 1 – Data Loss with emptyDir

Created a pod with emptyDir volume:

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: ephemeral-pod
spec:
  containers:
  - name: busybox
    image: busybox
    command: ["sh", "-c", "echo 'Written at: '$(date) > /data/message.txt && sleep 3600"]
    volumeMounts:
    - name: temp-storage
      mountPath: /data
  volumes:
  - name: temp-storage
    emptyDir: {}
  restartPolicy: Never
```

Verified data existed with kubectl exec, then deleted and recreated the pod.
The timestamp was different after recreation — old data was gone.

Key learning: emptyDir lives and dies with the Pod. It is useful for
temporary scratch space but not for anything that needs to persist.

---

## Task 2 – Creating a PersistentVolume (Static Provisioning)

```yaml
apiVersion: v1
kind: PersistentVolume
metadata:
  name: my-pv
spec:
  capacity:
    storage: 1Gi
  accessModes:
  - ReadWriteOnce
  persistentVolumeReclaimPolicy: Retain
  hostPath:
    path: /tmp/k8s-pv-data
```

### Field explanations:
- `capacity: 1Gi` — this PV offers 1GB of storage
- `ReadWriteOnce` — only one node can read/write at a time
- `Retain` — keep the data even after the PVC is deleted
- `hostPath` — uses a folder on the actual node (for learning only)

```bash
kubectl apply -f pv.yaml
kubectl get pv
```

Status showed Available — storage exists but no pod is using it yet.

### Access Modes:
| Mode | Short | Meaning |
|---|---|---|
| ReadWriteOnce | RWO | Read-write by a single node |
| ReadOnlyMany | ROX | Read-only by many nodes |
| ReadWriteMany | RWX | Read-write by many nodes |

---

## Task 3 – Creating a PersistentVolumeClaim

```yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: my-pvc
spec:
  accessModes:
  - ReadWriteOnce
  resources:
    requests:
      storage: 500Mi
```

Requesting 500Mi — Kubernetes matched this to the 1Gi PV since it fits
and the access modes match.

```bash
kubectl apply -f pvc.yaml
kubectl get pvc
kubectl get pv
```

Both showed Bound status. The VOLUME column in kubectl get pvc
showed my-pv — confirming they were linked together.

---

## Task 4 – Using PVC in a Pod

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: pvc-pod-1
spec:
  containers:
  - name: busybox
    image: busybox
    command: ["sh", "-c", "echo 'Pod 1 written at: '$(date) >> /data/message.txt && sleep 3600"]
    volumeMounts:
    - name: persistent-storage
      mountPath: /data
  volumes:
  - name: persistent-storage
    persistentVolumeClaim:
      claimName: my-pvc
  restartPolicy: Never
```

Wrote data from Pod 1, deleted the pod, created Pod 2 using the same PVC.
Checked the file from Pod 2 — it contained messages from BOTH pods.

Data survived the pod deletion because it was stored in the PV,
not inside the container filesystem.

Key difference:
- emptyDir — data dies with the pod
- PVC — data survives pod deletion and restarts

---

## Task 5 – StorageClasses

```bash
kubectl get storageclass
kubectl describe storageclass
```

On Minikube the default StorageClass is called standard.
StorageClasses define how storage is provisioned dynamically.

Key fields in a StorageClass:
- Provisioner — what creates the actual storage
- ReclaimPolicy — what happens to the PV when PVC is deleted
- VolumeBindingMode — when the PV gets created and bound

---

## Task 6 – Dynamic Provisioning

With dynamic provisioning developers only create PVCs.
The StorageClass automatically creates the PV — no manual PV needed.

```yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: dynamic-pvc
spec:
  accessModes:
  - ReadWriteOnce
  storageClassName: standard
  resources:
    requests:
      storage: 500Mi
```

```bash
kubectl apply -f dynamic-pvc.yaml
kubectl get pv
```

A new PV appeared automatically without manually creating one.
This is the real world approach — static provisioning is mostly
only used in on-premise environments.

Used the dynamic PVC in a pod and verified it worked correctly.

### Static vs Dynamic Provisioning:
| | Static | Dynamic |
|---|---|---|
| PV creation | Manual by admin | Automatic by StorageClass |
| Use case | On-premise, custom storage | Cloud environments |
| Flexibility | Less flexible | More flexible |

---

## Task 7 – Cleanup and Reclaim Policies

Deleted pods first, then deleted PVCs and observed:

```bash
kubectl delete pvc my-pvc dynamic-pvc
kubectl get pv
```

- my-pv (manual, Retain policy) — showed Released status, data kept
- dynamic PV (Delete policy) — completely gone, deleted automatically

### Reclaim Policies:
- Retain — PV stays after PVC deletion, data is preserved,
  admin must manually clean up and re-use the PV
- Delete — PV and underlying storage are automatically deleted
  when the PVC is deleted
- Recycle — deprecated, do not use

Deleted the remaining manual PV:
```bash
kubectl delete pv my-pv
```

---

## Key Takeaways

- Container storage is ephemeral by default — data dies with the pod
- PVs are cluster-wide storage resources independent of pods
- PVCs are namespaced requests for storage made by pods
- Kubernetes automatically matches PVCs to suitable PVs
- hostPath is only for learning — never use in production
- Static provisioning requires manual PV creation by an admin
- Dynamic provisioning automatically creates PVs via StorageClasses
- Retain policy keeps data after PVC deletion — good for databases
- Delete policy removes data automatically — good for temporary storage
- Always use PVCs in pod manifests, never reference PVs directly
- 
Proved this with an emptyDir pod:
- Wrote a timestamped message to /data/message.txt
- Deleted the pod and recreated it
- The file had a new timestamp — old data was completely gone

This is the problem Persistent Volumes solve.

---

## What are PVs and PVCs?

### PersistentVolume (PV)
A PV is actual storage that exists independently of any Pod.
It is a cluster-wide resource — not tied to any namespace.
Think of it like a physical hard drive plugged into the cluster.

### PersistentVolumeClaim (PVC)
A PVC is a request for storage made by a Pod.
It is namespaced — it belongs to a specific namespace.
Think of it like asking IT for a hard drive — you specify how much
you need and what access you need, and Kubernetes finds a matching PV.

### How they relate:
