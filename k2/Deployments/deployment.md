# 🚀 Manage Deployments

### `kubectl create deployment`

Creates a new Deployment with the specified number of replicas and container image.

```bash
kubectl create deployment <deployment-name> --replicas=3 --image=<image-name>
```

* `<deployment-name>` → Name of the Deployment
* `--replicas=3` → Runs 3 Pods
* `--image=<image-name>` → Container image to use

Example:

```bash
kubectl create deployment nginx --replicas=3 --image=nginx
```

---

### `kubectl scale deployment`

Changes the number of replicas in an existing Deployment.

```bash
kubectl scale deployment <deployment-name> --replicas=5
```

Example:

```bash
kubectl scale deployment nginx --replicas=5
```

This increases or decreases the number of running Pods.

---

### `kubectl set image`

Updates the container image used by a Deployment.

```bash
kubectl set image deployment/<deployment-name> <container-name>=<new-image>
```

Example:

```bash
kubectl set image deployment/nginx nginx=nginx:1.27
```

This starts a new rollout with the updated image.

---

### `kubectl rollout undo`

Rolls back a Deployment to its previous revision.

```bash
kubectl rollout undo deployment/<deployment-name>
```

Example:

```bash
kubectl rollout undo deployment/nginx
```

---

### `kubectl rollout history`

Shows the rollout history of a Deployment.

```bash
kubectl rollout history deployment/<deployment-name>
```

Example:

```bash
kubectl rollout history deployment/nginx
```

This helps you see the available Deployment revisions.

---

### `kubectl rollout undo --to-revision`

Rolls back a Deployment to a **specific revision**.

```bash
kubectl rollout undo deployment/<deployment-name> --to-revision=<revision-number>
```

Example:

```bash
kubectl rollout undo deployment/nginx --to-revision=2
```

Here, `2` is the revision number you want to restore.

---

### `kubectl get pods | grep <pod-name>`

get spesific pod.

```bash
kubectl get pods | grep <pod-name>
```


---

## 📋 Quick Reference

| Command                                    | Use                              |
| ------------------------------------------ | -------------------------------- |
| `kubectl create deployment ...`            | Create Deployment                |
| `kubectl scale deployment ...`             | Change number of replicas        |
| `kubectl set image ...`                    | Update container image           |
| `kubectl rollout undo ...`                 | Roll back to previous revision   |
| `kubectl rollout history ...`              | View rollout history             |
| `kubectl rollout undo ... --to-revision=2` | Roll back to a specific revision |
