<div align="center">

  <img src="./assets/kubernetes-banner.svg" width="100%" alt="Kubernetes Minikube Workflow" />

  <br/><br/>

  <p align="center">
    <strong>A modern, node-driven cheat sheet and declarative workflow manual for Kubernetes & Minikube.</strong>
  </p>

  <p align="center">
    <a href="https://kubernetes.io/"><img src="https://img.shields.io/badge/Kubernetes-141722?style=for-the-badge&logo=kubernetes&logoColor=FF6D5A" alt="Kubernetes" /></a>
    <a href="https://minikube.sigs.k8s.io/"><img src="https://img.shields.io/badge/Minikube-141722?style=for-the-badge&logo=minikube&logoColor=EA4B71" alt="Minikube" /></a>
    <a href="https://www.docker.com/"><img src="https://img.shields.io/badge/Docker-141722?style=for-the-badge&logo=docker&logoColor=white" alt="Docker" /></a>
    <img src="https://img.shields.io/badge/Status-Active-EA4B71?style=for-the-badge" alt="Status" />
  </p>

  <p align="center">
    <img src="https://readme-typing-svg.demolab.com?font=JetBrains+Mono&weight=600&size=15&duration=2000&pause=800&color=FF6D5A&center=true&vCenter=true&width=750&height=35&lines=minikube+start+--driver%3Ddocker;kubectl+apply+-f+deployment.yaml;kubectl+rollout+status+deployment%2Fapp-node;kubectl+get+pods+-o+wide" alt="Typing SVG" />
  </p>

</div>

---

## 🎯 Architecture Nodes

Like visual workflow automation, Kubernetes routes traffic and computes through structured, interconnected nodes:

| Node Type | Primitive | Purpose |
|:---|:---|:---|
| **Trigger Node** | `Ingress` / `Service` | Ingests incoming network traffic and maps routes to available target endpoints. |
| **Action Node** | `Deployment` / `Pod` | Executes containers, runs processes, and controls replica states. |
| **Data Node** | `PVC` / `ConfigMap` | Injects persistent block volumes, environment variables, and operational configurations. |
| **Control Node** | `Kube-Controller` | Continuously reconciles cluster actual state against desired declarative state. |

---

## ⚡ Cluster Initiation

```bash
# 1. Start local single-node cluster with container isolation
minikube start --driver=docker --cpus=4 --memory=4096

# 2. Inspect active control plane and nodes
kubectl cluster-info
kubectl get nodes -o wide

# 3. Open integrated visual dashboard
minikube dashboard