# 🌐 Kubernetes Services (Like a Load Balancer)

## What is a Service?

Pods are **ephemeral** — they get created and destroyed constantly (crashes, scaling, rolling updates, rescheduling). Every time a pod is recreated, it gets a **new internal IP address**. That makes it impossible for other apps to reliably talk to a pod directly.

A **Kubernetes Service** solves this by giving you a **stable, single endpoint** (a virtual IP) that sits in front of a group of pods. Kubernetes keeps track of which pods are currently alive and automatically routes traffic to them.

```
                 ┌─────────────────────────┐
   Client  ───▶  │   Service (Virtual IP)   │
                 │      10.98.147.95        │
                 └───────────┬─────────────┘
                              │ load-balances
             ┌────────────────┼────────────────┐
             ▼                ▼                ▼
        ┌─────────┐     ┌─────────┐      ┌─────────┐
        │  Pod 1  │     │  Pod 2  │      │  Pod 3  │
        │ 10.244  │     │ 10.244  │      │ 10.244  │
        │  .0.13  │     │  .0.11  │      │  .0.12  │
        └─────────┘     └─────────┘      └─────────┘
        (app=myapp)     (app=myapp)      (app=myapp)
```

A Service provides:
- 🔁 **Load balancing** — spreads traffic across all matching pods
- 🔍 **Service discovery** — other apps find it by a stable DNS name, not an IP
- 🚀 **Zero-downtime deployments** — new pods are added/removed from routing automatically

Services find their pods using **label selectors** (e.g. `app=myapp`) — any pod carrying that label gets included, regardless of which pod died or was created.

---

## Service Types

| Type | Scope | Description |
|------|-------|-------------|
| **ClusterIP** | Internal only | Default type. Creates a service reachable only *inside* the cluster on a cluster-internal IP. |
| **NodePort** | Internal + external (via node) | Like ClusterIP, but also opens the **same port on every node** in the cluster. Traffic hitting `<NodeIP>:<NodePort>` gets forwarded to the service. |
| **LoadBalancer** | External (cloud) | Provisions your cloud provider's native load balancer (AWS ELB, GCP LB, Azure LB, etc.) and points it at the service. Requires running on a supported cloud provider. |

```
ClusterIP                NodePort                    LoadBalancer
┌─────────┐          ┌───────────────┐          ┌──────────────────┐
│ Cluster │          │  External     │          │  Cloud Provider   │
│ Internal│          │  Client       │          │  (AWS/GCP/Azure)  │
│  Only   │          │     │         │          │        │          │
└────┬────┘          │     ▼         │          │        ▼          │
     │                │ Node:30500   │          │  LoadBalancer IP  │
     ▼                │     │        │          │        │          │
  Pods                │     ▼        │          │        ▼          │
                       │  Service     │          │     Service       │
                       │     │        │          │        │          │
                       │     ▼        │          │        ▼          │
                       │   Pods       │          │      Pods         │
                       └───────────────┘          └──────────────────┘
```

---

## Managing Services — Command Reference

### 1. List services

```bash
kubectl get svc              # Lists all services in the current namespace
kubectl get svc -A           # Lists all services across all namespaces
```

### 2. Describe a service

```bash
kubectl describe svc <service-name>   # Shows detailed info about a service
```

### 3. Expose a deployment as a service

```bash
kubectl expose deployment <deployment-name> \
  --port=<port> \
  --target-port=<container-port> \
  --type=<service_type>
```
- `--port` → the port the **service** listens on
- `--target-port` → the port the **container/pod** listens on
- `--type` → `ClusterIP` (default) | `NodePort` | `LoadBalancer`

### 4. Apply a service from YAML

```bash
kubectl apply -f <service-config.yaml>   # Creates or updates a service from a YAML file
```

### 5. Delete a service

```bash
kubectl delete svc <service-name>   # Deletes a specific service in the current namespace
```

### 6. Edit a service

```bash
kubectl edit svc <service-name>   # Edits an existing service configuration live
```

### 7. Get node IP addresses

```bash
kubectl get nodes -o wide   # Shows node IP addresses (needed for NodePort access)
```

### 8. Open a service in the browser (Minikube only)

```bash
minikube service <service-name>   # Launches the service in a browser using Minikube's IP + NodePort
```

---

## 📋 Quick Reference Table

| Action | Command |
|---|---|
| List services (current ns) | `kubectl get svc` |
| List services (all ns) | `kubectl get svc -A` |
| Describe a service | `kubectl describe svc <name>` |
| Expose a deployment | `kubectl expose deployment <name> --port=<p> --target-port=<tp> --type=<type>` |
| Apply from YAML | `kubectl apply -f <file.yaml>` |
| Delete a service | `kubectl delete svc <name>` |
| Edit a service | `kubectl edit svc <name>` |
| Get node IPs | `kubectl get nodes -o wide` |
| Open service in browser (minikube) | `minikube service <name>` |

---

## 🔬 Full Walkthrough (Real Session Output)

**Step 1 — Deploy the app**
```
C:\kubernetes\k4>kubectl apply -f my-deployment.yml
deployment.apps/myapp created
```

**Step 2 — Check the pods** (each gets its own internal pod IP)
```
C:\kubernetes\k4>kubectl get pods -o wide
NAME                     READY   STATUS    RESTARTS   AGE   IP            NODE       NOMINATED NODE   READINESS GATES
myapp-6d466cb9b9-tcdhf   1/1     Running   0          32s   10.244.0.13   minikube   <none>           <none>
myapp-6d466cb9b9-wpx8d   1/1     Running   0          32s   10.244.0.11   minikube   <none>           <none>
myapp-6d466cb9b9-z49hf   1/1     Running   0          32s   10.244.0.12   minikube   <none>           <none>
```

**Step 3 — Expose the deployment as a ClusterIP service (default type)**
```
C:\kubernetes\k4>kubectl expose deploy myapp --port=3000 --target-port=80
service/myapp exposed
```

**Step 4 — Verify the service**
```
C:\kubernetes\k4>kubectl get svc
NAME         TYPE        CLUSTER-IP     EXTERNAL-IP   PORT(S)    AGE
kubernetes   ClusterIP   10.96.0.1      <none>        443/TCP    3m47s
myapp        ClusterIP   10.98.147.95   <none>        3000/TCP   11s
```

**Step 5 — View pods and service together, with selector**
```
C:\kubernetes\k4>kubectl get pods,svc -o wide
NAME                         READY   STATUS    RESTARTS   AGE     IP            NODE       NOMINATED NODE   READINESS GATES
pod/myapp-6d466cb9b9-tcdhf   1/1     Running   0          3m27s   10.244.0.13   minikube   <none>           <none>
pod/myapp-6d466cb9b9-wpx8d   1/1     Running   0          3m27s   10.244.0.11   minikube   <none>           <none>
pod/myapp-6d466cb9b9-z49hf   1/1     Running   0          3m27s   10.244.0.12   minikube   <none>           <none>

NAME                 TYPE        CLUSTER-IP     EXTERNAL-IP   PORT(S)    AGE     SELECTOR
service/kubernetes   ClusterIP   10.96.0.1      <none>        443/TCP    5m16s   <none>
service/myapp        ClusterIP   10.98.147.95   <none>        3000/TCP   100s    app=myapp
```
Notice the `SELECTOR` column — `app=myapp` is exactly how the service knows which pods belong to it.

**Step 6 — Delete the ClusterIP service**
```
C:\kubernetes\k4>kubectl delete svc myapp
service "myapp" deleted from default namespace
```

**Step 7 — Re-expose it, this time as NodePort**
```
C:\kubernetes\k4>kubectl expose deploy myapp --port=5000 --target-port=80 --type=NodePort
service/myapp exposed
```

**Step 8 — Get node info (for NodePort access)**
```
C:\kubernetes\k4>kubectl get nodes
NAME       STATUS   ROLES           AGE   VERSION
minikube   Ready    control-plane   27h   v1.37.0
```

**Step 9 — List all Minikube services**
```
C:\kubernetes\k4>minikube service list
┌──────────────────────┬───────────────────────────┬──────────────┬─────┐
│      NAMESPACE       │           NAME            │ TARGET PORT  │ URL │
├──────────────────────┼───────────────────────────┼──────────────┼─────┤
│ default              │ kubernetes                │ No node port │     │
│ default              │ myapp                     │ 5000         │     │
│ kube-system          │ kube-dns                  │ No node port │     │
│ kubernetes-dashboard │ dashboard-metrics-scraper │ No node port │     │
│ kubernetes-dashboard │ kubernetes-dashboard      │ No node port │     │
└──────────────────────┴───────────────────────────┴──────────────┴─────┘
```

**Step 10 — Open the service in the browser**
```
C:\kubernetes\k4>minikube service myapp
┌───────────┬───────┬─────────────┬───────────────────────────┐
│ NAMESPACE │ NAME  │ TARGET PORT │            URL             │
├───────────┼───────┼─────────────┼───────────────────────────┤
│ default   │ myapp │ 5000        │ http://192.168.49.2:32243 │
└───────────┴───────┴─────────────┴───────────────────────────┘
🔗  Starting tunnel for service myapp.
┌───────────┬───────┬─────────────┬────────────────────────┐
│ NAMESPACE │ NAME  │ TARGET PORT │          URL           │
├───────────┼───────┼─────────────┼────────────────────────┤
│ default   │ myapp │             │ http://127.0.0.1:53651 │
└───────────┴───────┴─────────────┴────────────────────────┘
🎉  Opening service default/myapp in default browser...
❗  Because you are using a Docker driver on windows, the terminal needs to be open to run it.
✋  Stopping tunnel for service myapp.
```

> 💡 On Windows/macOS with the Docker driver, Minikube can't directly attach a NodePort to your host — so it opens a **tunnel** and keeps the terminal open to route traffic through to the container.

---

## 📄 Example YAML Manifests

### Deployment (`my-deployment.yml`)
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: myapp
spec:
  replicas: 3
  selector:
    matchLabels:
      app: myapp
  template:
    metadata:
      labels:
        app: myapp
    spec:
      containers:
      - name: myapp
        image: nginx
        resources:
          limits:
            memory: "128Mi"
            cpu: "500m"
        ports:
        - containerPort: 80
```

### Service (`my-service.yml`)
```yaml
apiVersion: v1
kind: Service
metadata:
  name: my-service
spec:
  selector:
    app: myapp
  type: NodePort
  ports:
  - port: 4000
    targetPort: 80
    nodePort: 30500
```

**How the fields line up:**

| YAML field | Meaning |
|---|---|
| `spec.selector.app: myapp` | Matches pods labeled `app: myapp` (must match the Deployment's pod labels) |
| `spec.type: NodePort` | Exposes the service on every node's IP at `nodePort` |
| `spec.ports.port` | Port the **Service** itself listens on internally (`4000`) |
| `spec.ports.targetPort` | Port on the **pod/container** traffic is forwarded to (`80`) |
| `spec.ports.nodePort` | Port opened on **every node** for external access (`30500`, must be in range 30000–32767) |

```
 External Client
       │
       ▼  http://<NodeIP>:30500
 ┌───────────────┐
 │   NodePort     │
 └───────┬────────┘
         │  service port 4000
         ▼
 ┌───────────────┐
 │   Service      │  (selector: app=myapp)
 └───────┬────────┘
         │  targetPort 80
         ▼
 ┌───────────────┐
 │   Pod(s)       │  containerPort: 80
 └───────────────┘
```

---

## 🧭 Service DNS Resolution

Kubernetes ships with a built-in DNS server — **kube-dns** (legacy) or **CoreDNS** (current default) — that gives every Service and Pod a resolvable DNS name inside the cluster.

Whenever a Service is created, Kubernetes automatically creates a DNS record for it, following this pattern:

```
<service-name>.<namespace>.svc.cluster.local
```

```
        ┌─────────────────────────────┐
        │        CoreDNS / kube-dns    │
        │  (cluster-internal resolver) │
        └───────────────┬──────────────┘
                         │  resolves
                         ▼
        myapp.default.svc.cluster.local
                         │
                         ▼
                  10.98.147.95  (Service ClusterIP)
                         │
                         ▼
                    Pod, Pod, Pod
```

**Example:** a pod in the `default` namespace can reach the `myapp` service either by:
- Short name (same namespace): `myapp`
- Fully qualified name (any namespace): `myapp.default.svc.cluster.local`

This is why hardcoding pod IPs is never necessary — apps should always talk to each other via **Service DNS names**, since those stay constant even as pods are replaced.

---

## ✅ Key Takeaways

- Pods are disposable; **Services are stable**. Always talk to Services, not pod IPs.
- **ClusterIP** = internal-only (default). **NodePort** = internal + reachable via `<NodeIP>:<Port>`. **LoadBalancer** = cloud-provider external LB.
- Services find pods via **label selectors** — matching labels is what wires a Service to its Pods.
- Every Service gets a DNS name automatically: `<service-name>.<namespace>.svc.cluster.local`.
- `kubectl expose` is the quick one-liner way to create a Service; `kubectl apply -f` is the reproducible, version-controlled way (preferred for real projects).