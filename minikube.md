# ☸️ Kubernetes

## 📥 Steps to Download

### Minikube

Download **Minikube** from the official website. Minikube is used to create and run a Kubernetes cluster locally on your computer.

### kubectl

Download **kubectl**, the command-line tool used to communicate with your Kubernetes cluster.

---

# 🚀 Minikube Commands

### ▶️ `minikube start`

Starts the Kubernetes cluster.

```bash
minikube start
```

Use this when you want to start working with Kubernetes.

---

### 🛑 `minikube stop`

Stops the Kubernetes cluster.

```bash
minikube stop
```

The cluster can be started again using `minikube start`.

---

### 🗑️ `minikube delete`

Deletes the Minikube cluster.

```bash
minikube delete
```

Use this when you want to completely remove the local cluster.

---

### 🔍 `minikube status`

Shows the current status of the Kubernetes cluster.

```bash
minikube status
```

It helps you check whether the cluster is running or stopped.

---

### 📊 `minikube dashboard`

Opens the Kubernetes Dashboard in your browser.

```bash
minikube dashboard
```

It provides a graphical interface for viewing and managing the cluster.

---

### ☸️ `minikube kubectl -- version`

Shows the Kubernetes version through Minikube's `kubectl`.

```bash
minikube kubectl -- version
```

Minikube can provide a compatible `kubectl` command for your cluster.

---

### ➕ `minikube node add`

Adds a new node to the Minikube cluster.

```bash
minikube node add
```

Useful when working with a multi-node Kubernetes cluster.

---

### ➕ `kubectl cluster-info`

Info of a cluster

```bash
kubectl cluster-info
```

Useful when working with a multi-node Kubernetes cluster.

---

### ➖ `minikube node delete <node-name>`

Removes a node from the Minikube cluster.

```bash
minikube node delete <node-name>
```

Example:

```bash
minikube node delete minikube-m02
```

Replace `<node-name>` with the name of the node you want to remove.

---

## 📋 Quick Summary

| Command                            | Use                      |
| ---------------------------------- | ------------------------ |
| `minikube start`                   | Start cluster            |
| `minikube stop`                    | Stop cluster             |
| `minikube delete`                  | Delete cluster           |
| `minikube status`                  | Check status             |
| `minikube dashboard`               | Open GUI dashboard       |
| `minikube kubectl -- version`      | Check Kubernetes version |
| `minikube node add`                | Add a node               |
| `minikube node delete <node-name>` | Delete a node            |

**Simple flow:**

```text
Download
   ↓
minikube start
   ↓
minikube status
   ↓
minikube dashboard
   ↓
Work with Kubernetes
   ↓
minikube stop
```
