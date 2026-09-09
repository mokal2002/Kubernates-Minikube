# 💾 Kubernetes Persistent Volumes

## 📑 Index

1. [Storage Requirements](#1-storage-requirements)
2. [Volumes vs Persistent Volumes](#2-volumes-vs-persistent-volumes)
3. [Types of Persistent Volumes](#3-types-of-persistent-volumes)
4. [PV and PVC — The Big Picture](#4-pv-and-pvc--the-big-picture)
5. [The Lifecycle of PVs and PVCs](#5-the-lifecycle-of-pvs-and-pvcs)
6. [PV and PVC — YAML Fields Explained](#6-pv-and-pvc--yaml-fields-explained)
7. [Examples of PV Configs](#7-examples-of-pv-configs)
8. [Access Modes](#8-access-modes)
9. [Dynamic Provisioning with StorageClass](#9-dynamic-provisioning-with-storageclass)
10. [Dynamically Provisioning PVs](#10-dynamically-provisioning-pvs)
11. [Dynamic Provisioning — Full Flow](#11-dynamic-provisioning--full-flow)
12. [Why Persistent Storage Matters — Postgres Example](#12-why-persistent-storage-matters--postgres-example)
13. [Full Working Example — Deployment + PV + PVC](#13-full-working-example--deployment--pv--pvc)
14. [Managing PVs, PVCs & Pods — Command Reference](#14-managing-pvs-pvcs--pods--command-reference)
15. [Key Takeaways](#15-key-takeaways)

---

## 1. Storage Requirements

Regular container storage disappears the moment a pod dies. Real applications (databases, file uploads, logs) need storage that behaves differently:

1. **Storage that doesn't depend on the pod lifecycle** — data must outlive the pod.
2. **Storage must be available on all nodes** — a pod could be rescheduled onto any node.
3. **Storage needs to survive even if the cluster crashes** — data shouldn't vanish with the cluster.

> ✅ To satisfy these three requirements, Kubernetes gives us **Persistent Volumes (PVs)**.

---

## 2. Volumes vs Persistent Volumes

| | **Volume** | **Persistent Volume (PV)** |
|---|---|---|
| Lifecycle | Tied to the **Pod** that owns it | Independent of any single Pod |
| Data survives pod deletion? | ❌ No | ✅ Yes |
| Scope | Exists only in the context of a specific Pod | Cluster-wide resource |
| Use case | Temp scratch space, sharing data between containers in the same pod | Databases, persistent app state |

**In plain words:**
- A **Volume** lets containers *inside the same Pod* safely share storage — but when the Pod terminates, that volume's data is gone.
- A **Persistent Volume** decouples storage from any specific Pod, so the data survives pod restarts, rescheduling, or deletion.

```
Volume (pod-scoped)                 Persistent Volume (cluster-scoped)
┌───────────┐                        ┌──────────────────────────┐
│    Pod    │                        │        Cluster            │
│  ┌─────┐  │                        │   ┌───────┐   ┌────────┐  │
│  │ vol │  │  ✂ pod dies            │   │  PV   │───│ Storage │  │
│  └─────┘  │  → data lost           │   └───┬───┘   └────────┘  │
└───────────┘                        │       │ survives pod death│
                                      │   ┌───▼───┐               │
                                      │   │  Pod  │  (can change) │
                                      │   └───────┘               │
                                      └──────────────────────────┘
```

---

## 3. Types of Persistent Volumes

| Type | Description | Best for |
|---|---|---|
| **hostPath** | Uses a directory on the **host machine's filesystem**. | Testing/development only — no replication or dynamic provisioning, so **not for production**. |
| **nfs** | Connects to a **Network File System (NFS)** mount — a central file server shared across clients. | Shared read/write access across multiple pods/nodes. |
| **csi** | Uses the **Container Storage Interface (CSI)** spec to integrate with external storage providers (e.g., AWS EBS, Azure Disk). | Production cloud environments. |

```
hostPath                nfs                          csi
┌─────────┐          ┌──────────────┐          ┌───────────────────┐
│  Node   │          │  NFS Server   │          │  Cloud Provider    │
│ ┌─────┐ │          │  (central     │          │  (AWS EBS, Azure   │
│ │ dir │ │          │   storage)    │          │   Disk, GCE PD)    │
│ └─────┘ │          └──────┬───────┘          └─────────┬─────────┘
└─────────┘                 │  shared by many nodes       │  via CSI driver
                             ▼                             ▼
                          Pods                            Pods
```

---

## 4. PV and PVC — The Big Picture

Two separate resources work together:

- **PersistentVolume (PV)** — a piece of storage in the cluster, provisioned by an **Administrator** (or dynamically by a StorageClass). It exists independently of any Pod.
- **PersistentVolumeClaim (PVC)** — a request for storage made by a **Developer**. Think of it as "I need 1Gi of ReadWriteOnce storage" — Kubernetes then matches (binds) it to a suitable PV.

```
        Admin                                    Developer
          │                                           │
          │ provision                                 │ create
          ▼                                            ▼
   ┌─────────────┐        bind          ┌─────────────┐
   │      PV      │◀────────────────────│     PVC      │
   └─────────────┘                       └──────┬──────┘
                                                  │ use
                                                  ▼
                                          ┌──────────────┐
                                          │  Pod (volume) │
                                          └──────────────┘
```

**Ownership split:**

```
                Administrator Owned
┌─────────────────────────────────────────────┐
│      Pool of Persistent Volumes               │
│   [NFS PV]  [iSCSI PV]  [NFS PV]  [GCE PV]    │
└─────────────────────────────────────────────┘
        Administrator registers PVs in the pool
- - - - - - - - - - - - - - - - - - - - - - - - -
                Developer Owned
┌─────────────────────────────────────────────┐
│  Developer → claims a PV from the pool         │
│           → references the claim in a Pod       │
│                                                  │
│   [claim] ─────────▶  Pod { uses: claim }       │
└─────────────────────────────────────────────┘
```

**Key idea:** Admins manage the *supply* of storage (PVs); Developers manage the *demand* (PVCs) — the two are matched automatically by Kubernetes.

---

## 5. The Lifecycle of PVs and PVCs

There are **4 stages**:

1. **Provisioning** — the PV is created and its storage is allocated using the selected driver (hostPath, nfs, csi, etc.).
2. **Binding** — Kubernetes automatically watches for new PVCs and binds them to matching PVs. ⚠️ **Each PV can only be bound to a single PVC at a time.**
3. **Using** — once a PVC is consumed by a Pod, the underlying PV enters "in use," actively providing storage to that application.
4. **Reclaiming** — when a user deletes the PVC, access to the PV is released, and the storage is "reclaimed" (behavior depends on the PV's `persistentVolumeReclaimPolicy`: `Retain`, `Delete`, or `Recycle`).

```
 Provisioning ──▶ Binding ──▶ Using ──▶ Reclaiming
     │                │           │           │
  PV created      PVC matches  Pod mounts   PVC deleted
  (storage        to a PV      the PVC's    → storage
  allocated)       (1:1)        volume        reclaimed
```

---

## 6. PV and PVC — YAML Fields Explained

### PersistentVolume
```yaml
apiVersion: v1
kind: PersistentVolume
metadata:
  name: demo-pv
spec:
  accessModes:
    - ReadWriteOnce
  capacity:
    storage: 1Gi
  storageClassName: standard
  hostPath:
    path: /tmp/demo-pv
```

### PersistentVolumeClaim
```yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: demo-pvc
spec:
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 1Gi
  storageClassName: standard
  volumeName: demo-pv
```

**Line-by-line — how the two connect:**

| PV field | PVC field | Relationship |
|---|---|---|
| `metadata.name: demo-pv` | `spec.volumeName: demo-pv` | The PVC explicitly requests **this exact PV** by name (static binding). |
| `spec.accessModes` | `spec.accessModes` | Must be **compatible** — the PVC's requested mode must be supported by the PV. |
| `spec.capacity.storage: 1Gi` | `spec.resources.requests.storage: 1Gi` | The PVC's requested size must be **≤** the PV's capacity. |
| `spec.storageClassName: standard` | `spec.storageClassName: standard` | Must **match** for binding to succeed. |

```
   demo-pv (PV)                    demo-pvc (PVC)
┌──────────────────┐          ┌──────────────────┐
│ capacity: 1Gi     │◀────────│ requests: 1Gi     │
│ accessModes: RWO  │  bind   │ accessModes: RWO  │
│ storageClass:      │        │ storageClass:      │
│   standard         │        │   standard         │
│ hostPath: /tmp/...  │        │ volumeName:        │
│                     │        │   demo-pv           │
└──────────────────┘          └──────────────────┘
```

---

## 7. Examples of PV Configs

### a) Persistent Volume for Google Persistent Disk
```yaml
apiVersion: v1
kind: PersistentVolume
metadata:
  name: gcp-pd-pv
spec:
  capacity:
    storage: 50Gi
  accessModes:
    - ReadWriteOnce
  gcePersistentDisk:
    pdName: my-gcp-disk
    fsType: ext4
  persistentVolumeReclaimPolicy: Delete
```

### b) Persistent Volume for CSI (Container Storage Interface)
```yaml
apiVersion: v1
kind: PersistentVolume
metadata:
  name: csi-pv
spec:
  capacity:
    storage: 5Gi
  accessModes:
    - ReadWriteOnce
  csi:
    driver: ebs.csi.aws.com
    volumeHandle: vol-0123456789abcdef
    fsType: ext4
  persistentVolumeReclaimPolicy: Delete
```

### c) Persistent Volume for HostPath
```yaml
apiVersion: v1
kind: PersistentVolume
metadata:
  name: hostpath-pv
spec:
  capacity:
    storage: 1Gi
  accessModes:
    - ReadWriteOnce
  hostPath:
    path: /data/hostpath   # Path on the host node
  persistentVolumeReclaimPolicy: Retain
```

**What changes between these three:** only the storage backend block (`gcePersistentDisk`, `csi`, or `hostPath`) — everything else (`capacity`, `accessModes`, `persistentVolumeReclaimPolicy`) follows the same pattern regardless of backend.

---

## 8. Access Modes

Kubernetes provides **three access modes** for PVs:

| Mode | Short | Meaning | Typical use |
|---|---|---|---|
| **ReadWriteOnce** | RWO | Mounted as read/write by a **single node** | Single-instance or sharded DB storage |
| **ReadOnlyMany** | ROX | Mounted as **read-only** by many nodes | Read replicas of a DB |
| **ReadWriteMany** | RWX | Mounted as read/write by **many nodes** | Logging, data aggregation, NFS |

```
RWO                     ROX                      RWX
┌──────┐               ┌──────┐                 ┌──────┐
│ Node │──write──▶ PV  │ Node │──read──▶ PV      │ Node │──r/w──▶ PV
└──────┘               ├──────┤                 ├──────┤        ▲
  (only 1 node          │ Node │──read──▶┘       │ Node │──r/w──┤
   can write)           └──────┘                 └──────┘        │
                                                    (many nodes, r/w)
```

---

## 9. Dynamic Provisioning with StorageClass

- A **StorageClass** lets you define different *types* of storage (SSD vs HDD) and their configuration (performance, zones, etc.) — without an admin having to pre-create every PV by hand.
- Each StorageClass specifies a **provisioner** that manages the actual underlying storage, usually tied to a specific cloud/on-prem backend.

**Common provisioners:**

| Provisioner | Backend |
|---|---|
| `kubernetes.io/aws-ebs` | Amazon Elastic Block Store (EBS) |
| `kubernetes.io/gce-pd` | Google Compute Engine Persistent Disk |
| `kubernetes.io/azure-disk` | Microsoft Azure Disk |
| `nfs` | Network File System (shared read/write) |
| `kubernetes.io/no-provisioner` | Local storage volumes |

---

## 10. Dynamically Provisioning PVs

Instead of an admin manually pre-creating a PV, you can let Kubernetes create one automatically. **The trick: simply omit `volumeName` from the PVC's spec.**

### PVC (no `volumeName` → triggers dynamic provisioning)
```yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: pvc-dynamic
spec:
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 1Gi
  storageClassName: standard
```

### Pod referencing the PVC
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: pvc-pod
spec:
  containers:
    - name: pvc-pod-container
      image: nginx:latest
      volumeMounts:
        - mountPath: /data
          name: data
  volumes:
    - name: data
      persistentVolumeClaim:
        claimName: pvc-dynamic
```

**Line-by-line:**
- `metadata.name: pvc-dynamic` → the PVC's own name.
- `spec.storageClassName: standard` → tells Kubernetes *which* StorageClass's provisioner should create the PV.
- No `volumeName` present → Kubernetes doesn't bind to an existing PV; it **creates a brand-new PV automatically**.
- In the Pod, `volumes[].persistentVolumeClaim.claimName: pvc-dynamic` → wires the Pod's volume to that PVC.
- `volumeMounts[].mountPath: /data` → where inside the **container** the volume appears.

---

## 11. Dynamic Provisioning — Full Flow

1. **Cluster Admin** sets up a persistent volume provisioner and configures a set of **StorageClasses**.
2. The **application developer** can see what classes of storage are supported by listing `StorageClass` objects.
3. The developer creates a **PersistentVolumeClaim** that references one of the StorageClasses, and a **Pod** that references the claim.
4. The **provisioner** creates the underlying storage and the **PersistentVolume**.
5. **Kubernetes binds** the claim to the newly created PV.
6. When the **Pod runs**, the volume's underlying storage is mounted into the pod's container(s).

```
 Cluster Admin                                          User
      │                                                    │
      ▼                                                    ▼
 StorageClass  ◀── developer lists ──  (sees what's available)
      │
      ▼
 Persistent Volume Provisioner
      │  (4) creates
      ▼
 Persistent Volume ◀────── Underlying Storage
      │
      │ (5) Kubernetes binds
      ▼
 Persistent Volume Claim ◀── (3) developer creates, references StorageClass
      │
      │ (6) mounted when pod runs
      ▼
     Pod { Volume }
```

---

## 12. Why Persistent Storage Matters — Postgres Example

Imagine a `postgres-pod` running inside a Node (`10.1.1.1`) alongside an `orders-pod-abc2`. If Postgres only wrote to the pod's local filesystem, all data would vanish the moment that pod restarts or is rescheduled to a different node.

Instead, the Postgres pod's data is backed by a **Persistent Volume**, which is:

- ✅ **Independent of the pod** — pod can die/restart, data stays.
- ✅ **Independent of the node** — pod can be rescheduled to any node in the cluster, and the PV follows.
- ✅ **Independent of the cluster** — for cloud-backed PVs (EBS, GCE PD, etc.), data can even survive a full cluster teardown.

```
Node (10.1.1.1)                              Cluster's Storage Pool
┌────────────────────────┐                  ┌──────────────────────┐
│ orders-pod-abc2 ───▶    │                  │  ⬡  ⬡  ⬡  ⬡  ⬡        │
│   postgres-pod           │──────volume─────▶│      ⬡  ⬡              │
│   (postgres container)   │                  └──────────────────────┘
└────────────────────────┘
       persistent solution:
       • independent of pod
       • independent of node
       • independent of cluster
```

---

## 13. Full Working Example — Deployment + PV + PVC

This is the complete, connected set of manifests: a **Deployment** that mounts a PVC-backed volume, the **PersistentVolume** that provides the storage, and the **PersistentVolumeClaim** that binds them together.

### Deployment (mounts the PVC as `/data`)
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
        volumeMounts:
        - mountPath: /data
          name: data
      volumes:
      - name: data
        persistentVolumeClaim:
          claimName: mypvc
```

### PersistentVolume
```yaml
apiVersion: v1
kind: PersistentVolume
metadata:
  name: mypv
spec:
  capacity:
    storage: 1Gi
  volumeMode: Filesystem
  accessModes:
    - ReadWriteOnce
  persistentVolumeReclaimPolicy: Recycle
  storageClassName: standard
  hostPath:
    path: /tmp/demo-pv
```

### PersistentVolumeClaim
```yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: mypvc
spec:
  resources:
    requests:
      storage: 1Gi
  volumeMode: Filesystem
  storageClassName: standard
  accessModes:
    - ReadWriteOnce
  volumeName: mypv
```

**How they connect, field by field:**

| Deployment | → | PVC | → | PV |
|---|---|---|---|---|
| `volumes[].persistentVolumeClaim.claimName: mypvc` | references | `metadata.name: mypvc` | | |
| | | `spec.volumeName: mypv` | binds to | `metadata.name: mypv` |
| | | `spec.storageClassName: standard` | must match | `spec.storageClassName: standard` |
| | | `spec.accessModes: [ReadWriteOnce]` | must be compatible | `spec.accessModes: [ReadWriteOnce]` |
| | | `spec.resources.requests.storage: 1Gi` | must be ≤ | `spec.capacity.storage: 1Gi` |
| `volumeMounts[].mountPath: /data` | mounted inside container | | | (backed by `hostPath.path: /tmp/demo-pv` on the node) |

```
Deployment "myapp" (3 replicas)
        │
        │ volumes: { persistentVolumeClaim.claimName: mypvc }
        ▼
   PVC "mypvc"  ── requests 1Gi, RWO, class=standard, volumeName=mypv
        │
        │ binds (1:1)
        ▼
   PV "mypv"    ── capacity 1Gi, RWO, class=standard
        │
        │ backed by
        ▼
   hostPath: /tmp/demo-pv  (on the node)
```

**Note on `persistentVolumeReclaimPolicy: Recycle`:** this policy scrubs the volume's data (`rm -rf` equivalent) and makes it available for a new claim again. It's deprecated in modern Kubernetes in favor of dynamic provisioning — `Retain` (keep data, manual cleanup) or `Delete` (auto-delete underlying storage) are preferred today.

---

## 14. Managing PVs, PVCs & Pods — Command Reference

All the basic `kubectl` commands you need day-to-day when working with Persistent Volumes, from creation to inspecting data **inside** the pod with `exec`.

### a) Create / Apply

```bash
kubectl apply -f pv.yaml            # Create or update a PersistentVolume from YAML
kubectl apply -f pvc.yaml           # Create or update a PersistentVolumeClaim from YAML
kubectl apply -f deployment.yaml    # Create or update the Deployment that mounts the PVC
```
```
persistentvolume/mypv created
persistentvolumeclaim/mypvc created
deployment.apps/myapp created
```

### b) List

```bash
kubectl get pv                      # List all PersistentVolumes (cluster-scoped)
kubectl get pvc                     # List PVCs in the current namespace
kubectl get pvc -A                  # List PVCs across all namespaces
kubectl get pv,pvc                  # List both together
```
```
NAME   CAPACITY   ACCESS MODES   RECLAIM POLICY   STATUS   CLAIM          STORAGECLASS   AGE
mypv   1Gi        RWO            Recycle          Bound    default/mypvc  standard       2m

NAME    STATUS   VOLUME   CAPACITY   ACCESS MODES   STORAGECLASS   AGE
mypvc   Bound    mypv     1Gi        RWO            standard       2m
```
> 💡 `STATUS: Bound` confirms the PVC successfully matched a PV. `Pending` means Kubernetes hasn't found (or dynamically created) a matching PV yet.

### c) Describe (deep inspect)

```bash
kubectl describe pv <pv-name>       # Full details of a PersistentVolume
kubectl describe pvc <pvc-name>     # Full details of a PersistentVolumeClaim, including binding + events
```
```
Name:            mypv
Labels:          <none>
Status:          Bound
Claim:           default/mypvc
Reclaim Policy:  Recycle
Access Modes:    RWO
Capacity:        1Gi
Source:
    Type:  HostPath
    Path:  /tmp/demo-pv
```

### d) Delete

```bash
kubectl delete pvc <pvc-name>       # Delete a claim (releases the PV per its reclaim policy)
kubectl delete pv <pv-name>         # Delete a PersistentVolume directly
```
```
persistentvolumeclaim "mypvc" deleted
persistentvolume "mypv" deleted
```
⚠️ Deleting a PVC that's still mounted by a running Pod will hang in `Terminating` until the Pod stops using it.

### e) Edit live

```bash
kubectl edit pv <pv-name>           # Edit a PV's manifest live (e.g. reclaim policy)
kubectl edit pvc <pvc-name>         # Edit a PVC's manifest live
```

### f) `kubectl exec` — run commands inside a Pod's container

This is how you jump **into** a running pod to check the actual mounted volume contents, run diagnostics, or interact with an app (e.g. `psql` into Postgres).

```bash
kubectl exec -it <pod-name> -- /bin/bash        # Open an interactive bash shell inside the pod
kubectl exec -it <pod-name> -- sh               # Use sh if bash isn't available (e.g. alpine images)
kubectl exec -it <pod-name> -c <container-name> -- /bin/bash   # Target a specific container in a multi-container pod
kubectl exec <pod-name> -- <command>            # Run a single one-off command (non-interactive)
kubectl exec <pod-name> -- ls -la /data         # Check what's actually inside the mounted volume
kubectl exec <pod-name> -- df -h                # Check disk usage inside the pod (confirms the volume is mounted)
```

**Flags explained:**
| Flag | Meaning |
|---|---|
| `-i` | Keep STDIN open (interactive) |
| `-t` | Allocate a pseudo-TTY (so you get a proper shell prompt) |
| `-c` | Choose which container to exec into, when the pod has more than one |
| `--` | Everything after this is the command to run inside the container |

**Example session — verifying the PVC-backed volume:**
```
$ kubectl exec -it myapp-6d466cb9b9-tcdhf -- /bin/bash
root@myapp-6d466cb9b9-tcdhf:/# ls -la /data
total 8
drwxr-xr-x 2 root root 4096 Sep  9 10:15 .
drwxr-xr-x 1 root root 4096 Sep  9 10:15 ..
root@myapp-6d466cb9b9-tcdhf:/# echo "hello" > /data/test.txt
root@myapp-6d466cb9b9-tcdhf:/# exit
```
If you delete this pod and a new replica takes over the same PVC, `/data/test.txt` will still be there — proving the storage really is persistent.

### g) Quick Reference Table

| Action | Command |
|---|---|
| Apply a PV/PVC/Deployment | `kubectl apply -f <file.yaml>` |
| List PVs | `kubectl get pv` |
| List PVCs (current ns) | `kubectl get pvc` |
| List PVCs (all ns) | `kubectl get pvc -A` |
| Describe a PV | `kubectl describe pv <name>` |
| Describe a PVC | `kubectl describe pvc <name>` |
| Delete a PVC | `kubectl delete pvc <name>` |
| Delete a PV | `kubectl delete pv <name>` |
| Edit a PV live | `kubectl edit pv <name>` |
| Edit a PVC live | `kubectl edit pvc <name>` |
| Shell into a pod | `kubectl exec -it <pod> -- /bin/bash` |
| Run one command in a pod | `kubectl exec <pod> -- <command>` |
| Exec into a specific container | `kubectl exec -it <pod> -c <container> -- /bin/bash` |
| List pods (to find pod name for exec) | `kubectl get pods` |

---

## 15. Key Takeaways

- **Volumes** die with their Pod; **Persistent Volumes** don't.
- **PV** = the actual storage resource (created by an admin or dynamically via StorageClass). **PVC** = a developer's *request* for storage, which Kubernetes binds to a matching PV.
- A PV can be bound to **only one PVC at a time**.
- Lifecycle: **Provisioning → Binding → Using → Reclaiming**.
- Three backend types: **hostPath** (dev/test only), **nfs** (shared network storage), **csi** (cloud-native block storage).
- Three access modes: **RWO** (single node r/w), **ROX** (many nodes read-only), **RWX** (many nodes r/w).
- **Static provisioning**: PVC sets `volumeName` to bind to a specific, pre-created PV.
- **Dynamic provisioning**: PVC omits `volumeName` and sets `storageClassName` — Kubernetes creates the PV automatically via the StorageClass's provisioner.
- Persistent storage is what makes stateful workloads (like databases) survive pod restarts, node failures, and rescheduling.