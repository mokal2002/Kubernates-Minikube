# ☸️ kubectl Commands

### `kubectl cluster-info`

Shows basic information about the Kubernetes cluster, including the address of the control plane and cluster services.

```bash
kubectl cluster-info
```

---

### `kubectl get namespaces`

Lists all namespaces available in the Kubernetes cluster.

```bash
kubectl get namespaces
```

Namespaces are used to organize and separate resources inside a cluster.

---

### `kubectl get nodes`

Lists all nodes in the Kubernetes cluster and shows their current status.

```bash
kubectl get nodes
```

A node is a machine that runs workloads in the Kubernetes cluster.

---

### `kubectl describe node <node-name>`

Shows detailed information about a specific node.

```bash
kubectl describe node <node-name>
```

Replace `<node-name>` with the actual node name. This can show node information, resources, conditions, and related events.
