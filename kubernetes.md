# Kubernetes Study Notes — Lessons 1 to 17

A beginner-friendly Kubernetes study guide covering the lessons learned so far.

---

# Lesson 1 — Why Kubernetes Exists

Docker is great for packaging and running containers, but once an application grows, several problems appear:

- What if a server crashes?
- What if traffic increases suddenly?
- What if you need to scale from 5 containers to 20?
- What if you deploy a bad version and need to roll back?
- What if containers move between machines?

Kubernetes solves these problems by managing containers across machines.

## Desired State

The most important idea:

> Kubernetes is a **desired-state system**.

You describe what you want:

```text
Backend version: v1
Replicas: 5
Port: 8080
```

Kubernetes continuously checks whether reality matches that desired state.

Example:

```text
Desired state:
5 backend Pods

Actual state:
4 backend Pods

Kubernetes creates 1 more Pod

Actual state:
5 backend Pods
```

## Self-Healing

If you tell Kubernetes:

```text
I always want 3 copies of my backend running.
```

and one crashes, Kubernetes creates or restarts another one so there are 3 again.

This is called **self-healing**.

---

# Lesson 2 — Docker Image vs Container

A Docker **image** is the packaged blueprint of your application.

Example:

```text
my-backend:v1
```

It can contain:

- Application code
- Runtime
- Libraries
- Dependencies
- Configuration defaults

A **container** is a running instance of an image.

Think:

```text
Class → Object
Image → Container
```

One image can create many containers:

```text
            my-api:v1
               |
       ┌───────┼───────┐
       ↓       ↓       ↓
Container  Container  Container
   #1         #2         #3
```

Important:

```text
1 image can run many containers.
```

Example:

```text
nginx:latest
```

Run 5 times:

```text
1 image
5 containers
```

---

# Lesson 3 — What is a Pod?

A **Pod** is the smallest unit Kubernetes normally manages.

Kubernetes usually does not manage a container directly.

Think:

```text
Kubernetes
    ↓
   Pod
    ↓
Container
```

Example:

```text
Pod: backend-pod
└── Container: my-backend:v1
```

Most commonly:

```text
1 Pod
  ↓
1 Container
```

But a Pod can contain multiple containers when they need to work closely together.

Example:

```text
Pod
├── Main application container
└── Helper container
```

If you want 3 copies of your backend, Kubernetes usually creates 3 Pods:

```text
Pod 1 → backend container
Pod 2 → backend container
Pod 3 → backend container
```

## Basic Pod YAML

```yaml
apiVersion: v1
kind: Pod

metadata:
  name: my-app

spec:
  containers:
    - name: my-container
      image: nginx
```

This means roughly:

> Create a Pod named `my-app` and run an `nginx` container inside it.

---

# Lesson 4 — What is a Node?

A **Node** is a machine where Kubernetes runs Pods.

A Node can be:

- Physical server
- Virtual machine
- Cloud VM

Example:

```text
Kubernetes Cluster
│
├── Node 1
│   ├── Pod A
│   └── Pod B
│
├── Node 2
│   ├── Pod C
│   └── Pod D
│
└── Node 3
    └── Pod E
```

Important hierarchy:

```text
Cluster
  ↓
Node
  ↓
Pod
  ↓
Container
```

A Node can run many Pods.

---

# Lesson 5 — Kubernetes Cluster and Control Plane

A **Kubernetes Cluster** is the complete Kubernetes environment.

It usually has:

```text
Kubernetes Cluster
│
├── Control Plane
│
└── Worker Nodes
    ├── Node 1
    ├── Node 2
    └── Node 3
```

## Control Plane

The Control Plane manages the cluster.

It decides things like:

- Which Node should run a Pod?
- Did a Pod crash?
- Should another Pod be created?
- Does actual state match desired state?

## Main Control Plane Components

### API Server

The API Server is the front door of Kubernetes.

```text
You
 ↓
kubectl
 ↓
API Server
 ↓
Kubernetes Cluster
```

Example:

```bash
kubectl get pods
```

### Scheduler

The Scheduler decides:

> Which Node should run this Pod?

Example:

```text
New Pod
   ↓
Scheduler
   ↓
Node 2
```

### Controller Manager

Controllers compare:

```text
Desired State
     VS
Actual State
```

Example:

```text
Desired = 3 Pods
Actual = 2 Pods
```

Kubernetes creates another Pod.

### etcd

`etcd` is Kubernetes' key-value database.

It stores important cluster state and configuration.

Examples:

- Pods
- Nodes
- Deployments
- Configuration
- Cluster state

## Summary

```text
Control Plane → manages the cluster
Worker Nodes  → run applications
Scheduler     → chooses Nodes
API Server    → receives requests
etcd          → stores cluster state
```

---

# Lesson 6 — What is kubectl?

`kubectl` is the command-line tool used to communicate with Kubernetes.

```text
You
 ↓
kubectl
 ↓
Kubernetes API Server
 ↓
Cluster
```

## Common Commands

Show Pods:

```bash
kubectl get pods
```

Show Nodes:

```bash
kubectl get nodes
```

Show Services:

```bash
kubectl get services
```

Describe a Pod:

```bash
kubectl describe pod <pod-name>
```

Show logs:

```bash
kubectl logs <pod-name>
```

Memory shortcut:

```text
kubectl get       → show resources
kubectl describe  → detailed information
kubectl logs      → application logs
```

---

# Lesson 7 — What is a Deployment?

In real projects, you usually do not create Pods directly.

Pods are temporary.

Instead, Kubernetes commonly uses a **Deployment**.

Think:

```text
Deployment
   ↓
manages
   ↓
Pods
```

Example:

```text
Deployment: backend
Desired replicas: 3

        ↓

Pod 1 → backend
Pod 2 → backend
Pod 3 → backend
```

If one Pod crashes, Kubernetes creates a replacement.

## Pod vs Deployment

```text
Pod
→ runs container(s)

Deployment
→ manages Pods
→ keeps desired number running
→ supports scaling
→ supports updates
→ supports rollback
```

## Basic Deployment YAML

```yaml
apiVersion: apps/v1
kind: Deployment

metadata:
  name: backend

spec:
  replicas: 3

  selector:
    matchLabels:
      app: backend

  template:
    metadata:
      labels:
        app: backend

    spec:
      containers:
        - name: backend
          image: myapp:v1
```

Important:

```yaml
replicas: 3
```

means:

> Keep 3 copies of the Pod running.

## Scaling

Example:

```bash
kubectl scale deployment backend --replicas=10
```

This changes the Deployment to 10 replicas.

---

# Lesson 8 — What is a ReplicaSet?

A **ReplicaSet** makes sure a specific number of Pods are running.

Relationship:

```text
Deployment
    ↓
ReplicaSet
    ↓
Pods
```

Example:

```text
Deployment: backend
      ↓
ReplicaSet
      ↓
 ┌────┼────┐
 ↓    ↓    ↓
Pod  Pod  Pod
```

If one Pod dies:

```text
Desired: 3
Actual: 2
```

The ReplicaSet creates a replacement.

Usually, you do not create ReplicaSets manually.

A Deployment creates and manages ReplicaSets for you.

Remember:

```text
Deployment
   ↓ manages
ReplicaSet
   ↓ manages
Pods
   ↓ contain
Containers
```

## Where do Nodes fit?

The management relationship:

```text
Deployment
    ↓
ReplicaSet
    ↓
Pods
```

The physical placement relationship:

```text
Cluster
   ↓
Worker Nodes
   ↓
Pods
```

Example:

```text
You create Deployment
        ↓
Deployment
        ↓
ReplicaSet
        ↓
Creates 3 Pods
        ↓
Scheduler decides where they run
        ↓

Node 1       Node 2
├── Pod A    └── Pod C
└── Pod B
```

Important:

```yaml
replicas: 3
```

means **3 Pods**, not 3 Nodes.

A Node can run many Pods.

---

# Lesson 9 — What is a Service?

Pods are temporary, and Pod IP addresses can change.

Example:

```text
Pod 1 → 10.0.0.21
Pod 2 → 10.0.0.37
Pod 3 → 10.0.0.52
```

If Pod 1 dies, a replacement may get a different IP.

So applications should usually talk to a **Service**, not directly to Pod IPs.

```text
Frontend
   ↓
Backend Service
   ↓
┌──┼──┐
↓  ↓  ↓
P1 P2 P3
```

A Service gives a stable network endpoint.

## Deployment vs Service

```text
Deployment
→ manages Pods
```

```text
Service
→ provides network access to Pods
```

---

# Lesson 10 — Labels and Selectors

Services use **labels** and **selectors** to find Pods.

Example Pod labels:

```text
Pod 1 → app=backend
Pod 2 → app=backend
Pod 3 → app=backend

Pod 4 → app=frontend
```

Service selector:

```yaml
selector:
  app: backend
```

The Service sends traffic only to matching Pods.

```text
                Service
           selector: app=backend
                    |
          ┌─────────┼─────────┐
          ↓         ↓         ↓
       Pod 1      Pod 2      Pod 3
     app=backend app=backend app=backend

       Pod 4
     app=frontend
          ✗
```

## Deployment + Service Example

Deployment:

```yaml
apiVersion: apps/v1
kind: Deployment

metadata:
  name: backend

spec:
  replicas: 3

  selector:
    matchLabels:
      app: backend

  template:
    metadata:
      labels:
        app: backend

    spec:
      containers:
        - name: backend
          image: myapp:v1
          ports:
            - containerPort: 8080
```

Service:

```yaml
apiVersion: v1
kind: Service

metadata:
  name: backend-service

spec:
  selector:
    app: backend

  ports:
    - port: 80
      targetPort: 8080
```

Important connection:

```text
Pod label
app: backend

       ↕

Service selector
app: backend
```

## port vs targetPort

```yaml
port: 80
targetPort: 8080
```

Think:

```text
Service Port      Container Port
     80    ───────→    8080
```

Frontend can use:

```text
backend-service:80
```

The Service forwards requests to:

```text
Pod:8080
```

---

# Lesson 11 — Types of Kubernetes Services

Three important Service types:

```text
ClusterIP
NodePort
LoadBalancer
```

## ClusterIP

Default Service type.

Used for communication **inside the cluster**.

```text
Frontend Pod
     ↓
Backend Service
type: ClusterIP
     ↓
Backend Pods
```

Think:

```text
ClusterIP = internal access
```

## NodePort

Exposes the Service using a port on Worker Nodes.

Example:

```text
Your Browser
     ↓
Node IP : 30080
     ↓
NodePort Service
     ↓
Backend Pods
```

Example YAML:

```yaml
spec:
  type: NodePort

  selector:
    app: backend

  ports:
    - port: 80
      targetPort: 8080
      nodePort: 30080
```

Access example:

```text
http://NODE-IP:30080
```

## LoadBalancer

Commonly used in cloud environments.

```text
Internet
   ↓
External Load Balancer
   ↓
Kubernetes Service
   ↓
┌────────┼────────┐
↓        ↓        ↓
Pod 1   Pod 2   Pod 3
```

Think:

```text
LoadBalancer = external access through a load balancer
```

## Easy Memory Trick

```text
ClusterIP
→ Inside cluster

NodePort
→ Node IP + port

LoadBalancer
→ External load balancer
```

---

# Lesson 12 — ConfigMap and Secret

Applications need configuration.

Example normal configuration:

```text
APP_ENV=production
API_URL=https://api.example.com
LOG_LEVEL=info
```

Use a **ConfigMap** for normal, non-sensitive configuration.

## ConfigMap Example

```yaml
apiVersion: v1
kind: ConfigMap

metadata:
  name: backend-config

data:
  APP_ENV: production
  LOG_LEVEL: info
```

Concept:

```text
ConfigMap
   ↓
configuration
   ↓
Pod
   ↓
Container
```

## Secret

Use a **Secret** for sensitive values.

Examples:

```text
DATABASE_PASSWORD
API_KEY
TOKEN
DB_USERNAME
```

Example:

```yaml
apiVersion: v1
kind: Secret

metadata:
  name: backend-secret

type: Opaque

stringData:
  DB_USERNAME: admin
  DB_PASSWORD: supersecret
```

## ConfigMap vs Secret

```text
ConfigMap
→ normal configuration

Secret
→ sensitive configuration
```

Examples:

```text
APP_ENV=production       → ConfigMap
LOG_LEVEL=debug          → ConfigMap
DATABASE_HOST=db         → ConfigMap

DATABASE_PASSWORD=xyz    → Secret
API_KEY=abc123           → Secret
```

Important note:

Kubernetes Secrets are not automatically secure just because they are called Secrets. Production clusters need proper Secret protection and access control.

---

# Lesson 13 — Volumes and Persistent Storage

Data stored only inside a container can disappear when the container or Pod is replaced.

Example:

```text
Pod 1 ❌
  /data/users.db

        ↓ replacement

Pod 2 ✅
  /data/users.db ???
```

For important data, use persistent storage.

## Volume

A Volume gives containers storage beyond their normal container filesystem.

```text
Pod
├── Container
│      ↓
│   /data
│      ↓
└── Volume
```

Not every Volume is permanent.

## PersistentVolume — PV

A **PersistentVolume (PV)** represents persistent storage.

The storage may come from:

- Cloud disk
- Network storage
- NFS
- Local disk

Concept:

```text
PersistentVolume
       ↓
Actual Storage
```

## PersistentVolumeClaim — PVC

Applications usually request storage using a **PersistentVolumeClaim (PVC)**.

Think:

```text
Pod
 ↓
PVC
 ↓
PV
 ↓
Real storage
```

Analogy:

```text
PV  = available apartment
PVC = your request for an apartment
Pod = person who wants to live there
```

## PostgreSQL Example

```text
PostgreSQL Pod
      ↓
mounts
      ↓
PVC: postgres-storage
      ↓
PV
      ↓
Disk
```

If the Pod dies:

```text
Postgres Pod 1 ❌
       ↓
     PVC
       ↓
     Disk
       ↑
       │
Postgres Pod 2 ✅
```

The replacement Pod can reconnect to the same data.

## Example PVC

```yaml
apiVersion: v1
kind: PersistentVolumeClaim

metadata:
  name: postgres-data

spec:
  accessModes:
    - ReadWriteOnce

  resources:
    requests:
      storage: 10Gi
```

Pod reference:

```yaml
volumes:
  - name: database-storage
    persistentVolumeClaim:
      claimName: postgres-data
```

Mount it:

```yaml
volumeMounts:
  - name: database-storage
    mountPath: /var/lib/postgresql/data
```

---

# Lesson 14 — Deployment vs StatefulSet

If PostgreSQL runs inside Kubernetes, then PostgreSQL runs inside a Pod.

Example:

```text
Kubernetes Cluster
      ↓
PostgreSQL Pod
      ↓
PostgreSQL Container
```

But PostgreSQL does not have to run inside Kubernetes.

For example, if PostgreSQL is hosted in AWS RDS:

```text
Kubernetes Cluster
│
├── Backend Pod
│      ↓
│   Backend app
│
└──────────────→ AWS RDS PostgreSQL
```

Then you do **not** need a PostgreSQL Pod.

## Deployment

A Deployment is best for applications where Pods are interchangeable.

Examples:

```text
Frontend
Backend API
Workers
Go API
```

If Pod B dies:

```text
Pod B ❌
   ↓
Pod D ✅
```

You usually do not care that the replacement has a different identity.

## StatefulSet

A **StatefulSet** is for workloads that need stable identity and often persistent storage.

Example:

```text
StatefulSet: postgres

postgres-0
postgres-1
postgres-2
```

If:

```text
postgres-0
```

is recreated, it keeps the stable identity:

```text
postgres-0
```

Beginner rule:

```text
Stateless app
→ Deployment

Stateful app
→ StatefulSet
```

Examples:

```text
Frontend      → Deployment
Backend API   → Deployment
Worker        → Deployment

PostgreSQL    → StatefulSet
MongoDB       → often StatefulSet
Redis         → sometimes StatefulSet
```

---

# Lesson 15 — Namespaces

A **Namespace** divides one Kubernetes cluster into logical groups.

Example:

```text
Kubernetes Cluster
│
├── Namespace: dev
│   ├── backend
│   └── frontend
│
├── Namespace: staging
│   ├── backend
│   └── frontend
│
└── Namespace: production
    ├── backend
    └── frontend
```

You can have the same resource name in different namespaces:

```text
dev/backend
production/backend
```

These are different resources.

## Useful Commands

Show namespaces:

```bash
kubectl get namespaces
```

Short form:

```bash
kubectl get ns
```

Show Pods in production:

```bash
kubectl get pods -n production
```

Show Pods in all namespaces:

```bash
kubectl get pods -A
```

Important:

```text
Namespace ≠ Cluster
```

You can have:

```text
1 Cluster
├── dev namespace
├── staging namespace
└── production namespace
```

---

# Lesson 16 — Ingress

Ingress is commonly used to route HTTP/HTTPS traffic.

Suppose:

```text
example.com      → frontend
example.com/api  → backend
```

Ingress can route requests based on hostname or URL path.

Example:

```text
Internet
   ↓
Ingress
   ├── /      → frontend-service → frontend Pods
   └── /api   → backend-service  → backend Pods
```

## Simplified Ingress YAML

```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress

metadata:
  name: my-ingress

spec:
  rules:
    - host: example.com
      http:
        paths:
          - path: /api
            pathType: Prefix
            backend:
              service:
                name: backend-service
                port:
                  number: 80
```

Important flow:

```text
Internet
   ↓
Ingress
   ↓
Service
   ↓
Pod
   ↓
Container
```

## Ingress vs Service

```text
Service
→ sends traffic to Pods
```

```text
Ingress
→ routes external HTTP/HTTPS traffic to Services
```

Important:

An Ingress resource usually needs an **Ingress Controller** that actually performs the routing.

---

# Lesson 17 — Liveness and Readiness Probes

A Pod can be `Running` while the application inside it is unhealthy.

Example:

```text
Pod: Running ✅
Container: Running ✅
Go API: frozen ❌
```

Kubernetes uses **probes** to check application health.

Two important probes:

```text
Liveness Probe
Readiness Probe
```

## Liveness Probe

A liveness probe asks:

> Is the application still alive?

Example:

```text
GET /health
```

If the liveness probe keeps failing, Kubernetes may restart the container.

```text
Liveness fails
      ↓
Kubernetes restarts container
      ↓
Application starts again
```

Remember:

```text
Liveness
→ Is the app alive?
→ Failure may restart the container
```

## Readiness Probe

A readiness probe asks:

> Is the application ready to receive traffic?

Example:

A backend container starts, but needs time to:

- Connect to PostgreSQL
- Load configuration
- Initialize cache

During that time, you do not want the Service sending traffic to it.

```text
Service
   ↓
Pod
   ↓
Not Ready ❌
```

Once readiness succeeds:

```text
Readiness succeeds ✅
        ↓
Service can send traffic
```

Remember:

```text
Readiness
→ Is the app ready?
→ Failure removes the Pod from Service traffic
```

## Example Probe YAML

```yaml
livenessProbe:
  httpGet:
    path: /health
    port: 8080
  initialDelaySeconds: 10
  periodSeconds: 5
```

```yaml
readinessProbe:
  httpGet:
    path: /ready
    port: 8080
  initialDelaySeconds: 5
  periodSeconds: 5
```

Concept:

```text
/health → "Am I alive?"
/ready  → "Can I serve requests?"
```

Example Go handlers:

```go
http.HandleFunc("/health", func(w http.ResponseWriter, r *http.Request) {
    w.WriteHeader(http.StatusOK)
})

http.HandleFunc("/ready", func(w http.ResponseWriter, r *http.Request) {
    // Check important dependencies
    w.WriteHeader(http.StatusOK)
})
```

---

# Kubernetes Mental Model — Lessons 1 to 17

```text
Kubernetes Cluster
│
├── Control Plane
│   ├── API Server
│   ├── Scheduler
│   ├── Controller Manager
│   └── etcd
│
└── Worker Nodes
    ├── Pod
    │   └── Container
    └── Pod
        └── Container
```

Management relationship:

```text
Deployment
   ↓
ReplicaSet
   ↓
Pods
   ↓
Containers
```

Networking:

```text
Internet
   ↓
Ingress
   ↓
Service
   ↓
Pods
```

Configuration:

```text
ConfigMap → normal configuration
Secret    → sensitive configuration
```

Storage:

```text
Pod
 ↓
PVC
 ↓
PV
 ↓
Persistent Storage
```

Workload choice:

```text
Stateless app → Deployment
Stateful app  → StatefulSet
```

Health:

```text
Liveness  → should Kubernetes restart it?
Readiness → should it receive traffic?
```

---

# Important kubectl Commands Learned

```bash
kubectl get pods
kubectl get nodes
kubectl get services
kubectl get replicasets
kubectl get rs
kubectl get namespaces
kubectl get ns

kubectl get pods -n production
kubectl get pods -A

kubectl describe pod <pod-name>
kubectl logs <pod-name>

kubectl scale deployment backend --replicas=10
```

---

# Quick Revision

## Core hierarchy

```text
Cluster
  ↓
Node
  ↓
Pod
  ↓
Container
```

## Management hierarchy

```text
Deployment
  ↓
ReplicaSet
  ↓
Pod
  ↓
Container
```

## Networking

```text
Ingress
  ↓
Service
  ↓
Pod
```

## Storage

```text
Pod
 ↓
PVC
 ↓
PV
 ↓
Disk
```

## Configuration

```text
ConfigMap → normal values
Secret    → passwords / tokens / credentials
```

## Probes

```text
Liveness  → restart if unhealthy
Readiness → stop sending traffic until ready
```
