# 🗂️ Managing Kubernetes Namespaces with `kubectl`

Namespaces let you split a single cluster into multiple virtual clusters — handy for separating teams, environments (dev/staging/prod), or projects.

```
        ┌─────────────────────────────┐
        │       Kubernetes Cluster    │
        │  ┌────────┐   ┌────────┐    │
        │  │  dev   │   │  prod  │    │
        │  │🟢🟢🟢 │  │🔵🔵🔵 │    │
        │  └────────┘   └────────┘    │
        │       ┌────────┐            │
        │       │ staging│            │
        │       │  🟡🟡 │            │
        │       └────────┘            │
        └─────────────────────────────┘
```

---

## 1. List all namespaces

```bash
kubectl get namespaces
# or shorthand
kubectl get ns
```

**Output:**
```
NAME              STATUS   AGE
default           Active   45d
kube-system       Active   45d
kube-public       Active   45d
kube-node-lease   Active   45d
dev               Active   12d
```

---

## 2. Create a namespace

```bash
kubectl create namespace <namespace-name>
```

**Example:**
```bash
kubectl create namespace dev
```
```
namespace/dev created
```

---

## 3. Delete a namespace

```bash
kubectl delete namespace <namespace-name>
```

**Example:**
```bash
kubectl delete namespace dev
```
```
namespace "dev" deleted
```

⚠️ **Warning:** This deletes **every resource** inside that namespace (pods, services, configmaps, etc.) — it cascades.

```
   💣  delete namespace "dev"
    │
    ▼
   ⏳ Terminating...
    │
    ▼
   🗑️  pods, svc, cm, secrets... all gone
```

---

## 4. List pods in a specific namespace

```bash
kubectl get pods -n <namespace-name>
```

**Example:**
```bash
kubectl get pods -n dev
```
```
NAME                        READY   STATUS    RESTARTS   AGE
frontend-6b7f9c9d8f-xk2lp   1/1     Running   0          3h
backend-5d8f7c6b9d-mz9qw    1/1     Running   0          3h
```

---

## 5. Set a default namespace for your current context

Avoid typing `-n <namespace>` every time by switching your active context's default namespace.

```bash
kubectl config set-context --current --namespace=<namespace-name>
```

**Example:**
```bash
kubectl config set-context --current --namespace=dev
```
```
Context "minikube" modified.
```

Now `kubectl get pods` alone will only show pods from `dev`.

---

## 6. List pods across all namespaces

```bash
kubectl get pods --all-namespaces
# or shorthand
kubectl get pods -A
```

**Output:**
```
NAMESPACE     NAME                        READY   STATUS    RESTARTS   AGE
dev           frontend-6b7f9c9d8f-xk2lp   1/1     Running   0          3h
kube-system   coredns-565d847f94-2j9pk    1/1     Running   0          45d
kube-system   etcd-minikube               1/1     Running   0          45d
prod          api-79c9f8d5b7-nm4kt        1/1     Running   0          10d
```

---

## 7. Inspect a namespace in detail

```bash
kubectl describe namespace <namespace-name>
```

**Example:**
```bash
kubectl describe namespace dev
```
```
Name:         dev
Labels:       kubernetes.io/metadata.name=dev
Annotations:  <none>
Status:       Active

No resource quota.
No LimitRange resource.
```

---

## 8. Edit a namespace live

Opens the namespace's manifest (e.g., labels, annotations) in your default editor.

```bash
kubectl edit namespace <namespace-name>
```

```
   📝 opens YAML in $EDITOR
    │
    ▼
   ✏️  add labels/annotations
    │
    ▼
   💾 save → kubectl applies the patch instantly
```

---

## 9. Get ALL resources across ALL namespaces

The nuclear "show me everything" command.

```bash
kubectl get all --all-namespaces
```

**Output (trimmed):**
```
NAMESPACE   NAME                            READY   STATUS    RESTARTS   AGE
dev         pod/frontend-6b7f9c9d8f-xk2lp   1/1     Running   0          3h
kube-system pod/coredns-565d847f94-2j9pk    1/1     Running   0          45d

NAMESPACE     NAME                 TYPE        CLUSTER-IP   PORT(S)   AGE
default       service/kubernetes   ClusterIP   10.96.0.1    443/TCP   45d
dev           service/frontend     ClusterIP   10.96.5.22   80/TCP    3h

NAMESPACE   NAME                       READY   UP-TO-DATE   AVAILABLE
dev         deployment.apps/frontend   2/2     2            2
```

---

## 📋 Quick Reference Table

| Action                          | Command                                                        |
|----------------------------------|----------------------------------------------------------------|
| List namespaces                  | `kubectl get ns`                                                |
| Create namespace                 | `kubectl create namespace <name>`                               |
| Delete namespace                 | `kubectl delete namespace <name>`                               |
| Pods in a namespace              | `kubectl get pods -n <name>`                                    |
| Set default namespace            | `kubectl config set-context --current --namespace=<name>`       |
| Pods in all namespaces           | `kubectl get pods -A`                                           |
| Describe a namespace             | `kubectl describe namespace <name>`                             |
| Edit a namespace                 | `kubectl edit namespace <name>`                                 |
| All resources, all namespaces    | `kubectl get all --all-namespaces`                               |

---

### 💡 Pro tip
Combine `kubectl config set-context` with tools like [`kubectx`/`kubens`](https://github.com/ahmetb/kubectx) for fast namespace switching without typing the full command every time.