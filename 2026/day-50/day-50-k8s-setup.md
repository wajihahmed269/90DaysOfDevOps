Day 50 – Kubernetes Architecture and Cluster Setup
Task 1: The Kubernetes Story
Docker solved packaging and running containers on a single machine. But when you need hundreds of containers across dozens of servers — deciding where to run them, restarting them when they crash, scaling them up under load, rolling out updates without downtime — Docker alone has no answer for that. You need something to orchestrate the whole thing.
Kubernetes was built by Google, based on their internal system called Borg which had been running containerized workloads at Google-scale for years. They open-sourced it in 2014 and donated it to the CNCF. The name comes from the Greek word for "helmsman" or "pilot" — the person steering the ship.
Task 2: Kubernetes Architecture
                        CONTROL PLANE
┌─────────────────────────────────────────────────────┐
│                                                     │
│   ┌─────────────┐     ┌──────┐     ┌─────────────┐ │
│   │  API Server │────▶│ etcd │     │  Scheduler  │ │
│   │ (front door)│     │ (DB) │     │(picks nodes)│ │
│   └──────┬──────┘     └──────┘     └─────────────┘ │
│          │                                          │
│   ┌──────▼────────────┐                             │
│   │ Controller Manager│                             │
│   │ (watches & fixes) │                             │
│   └───────────────────┘                             │
└─────────────────────────────────────────────────────┘
                        │
            ┌───────────┼───────────┐
            │           │           │
       WORKER NODE  WORKER NODE  WORKER NODE
    ┌──────────────┐
    │  kubelet     │  ← talks to API server, manages pods
    │  kube-proxy  │  ← handles network rules
    │  containerd  │  ← actually runs containers
    │  [Pod][Pod]  │
    └──────────────┘
What happens when you run kubectl apply -f pod.yaml:

kubectl sends the YAML to the API Server via HTTP
API Server validates it and writes the desired state to etcd
The Scheduler notices an unscheduled pod and picks a node based on available resources
The kubelet on that node sees the assignment, tells the container runtime to pull the image and start the container
kubelet reports pod status back to the API Server, which writes it to etcd

If the API Server goes down: nothing new can be created, updated, or deleted. Existing pods keep running since kubelet manages them locally, but the cluster is essentially blind and unmanageable.
If a worker node goes down: the Controller Manager detects the node is unreachable, marks its pods as failed, and reschedules them on healthy nodes — but only if those pods are managed by a Deployment or ReplicaSet, not standalone pods.
Task 4: Cluster Setup
Chose: kind
Reason: already have Docker running, kind spins up a cluster in under a minute using containers as nodes. No VM overhead, no extra drivers. Fast enough for daily practice.
bashkind create cluster --name devops-cluster
kubectl cluster-info
kubectl get nodes
Expected output:
NAME                          STATUS   ROLES           AGE   VERSION
devops-cluster-control-plane  Ready    control-plane   60s   v1.29.x
Task 5: Cluster Exploration
bashkubectl get namespaces
# default, kube-node-lease, kube-public, kube-system

kubectl get pods -n kube-system
PodWhat it doesetcdStores all cluster state — the single source of truthkube-apiserverEvery kubectl command hits this firstkube-schedulerDecides which node gets a new podkube-controller-managerRuns all the control loops — node controller, replication controller, etc.corednsDNS for the cluster — how pods find each other by namekube-proxySets up iptables/ipvs rules on each node for pod networkingkindnet (kind-specific)The CNI plugin that handles pod-to-pod networking in kind
Task 6: Kubeconfig
A kubeconfig is a YAML file that stores cluster connection details — the server address, authentication credentials, and which cluster/namespace to use by default. It's what allows kubectl to know which cluster to talk to when you have multiple.
Stored at: ~/.kube/config
bashkubectl config current-context   # devops-cluster
kubectl config get-contexts       # lists all clusters kubectl knows about
kubectl config view               # prints the full kubeconfig
