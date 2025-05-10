# Kubernetes & AWS EKS: Complete Overview and Deployment Flow

This README provides a detailed yet clear understanding of Kubernetes architecture and how AWS EKS (Elastic Kubernetes Service) simplifies deploying and managing Kubernetes clusters. It includes real-world deployment flow, how components communicate, and scaling concepts.

---

## 📌 What is Kubernetes?
Kubernetes (K8s) is an open-source platform for automating:
- Deployment
- Scaling
- Management of containerized applications

It manages workloads through objects like **Pods**, **Deployments**, and **Services**.

---

## 🔹 Kubernetes Architecture (Self-Managed)

### Control Plane Components:
- **API Server**: Entry point for all control plane commands (via `kubectl`)
- **Scheduler**: Assigns pods to nodes
- **Controller Manager**: Maintains the cluster's desired state
- **etcd**: Stores all cluster data (like a database)

### Node Components:
- **kubelet**: Talks to control plane, runs containers
- **kube-proxy**: Handles network routing
- **Container Runtime**: Docker, containerd, etc.

---

## ☁️ What is Amazon EKS?

Amazon EKS (Elastic Kubernetes Service) is a **fully managed Kubernetes control plane** provided by AWS.

### Key Features:
- AWS handles the provisioning and management of the **control plane**
- Automatically replicates control plane components across **3 Availability Zones (AZs)** for high availability
- You only manage the **worker nodes** (EC2 instances)

### Why EKS?
- No need to manage master nodes manually
- Integrated IAM, VPC, ELB, CloudWatch, etc.
- Simplifies production-grade Kubernetes setup

---

## 🛠️ Tools You Need for EKS

### 1. `eksctl`: Cluster provisioning tool
```bash
curl -s --location "https://github.com/eksctl-io/eksctl/releases/latest/download/eksctl_$(uname -s)_amd64.tar.gz" \
  | tar xz -C /tmp
sudo mv /tmp/eksctl /usr/local/bin
```

### 2. `kubectl`: Kubernetes CLI
```bash
curl -LO "https://dl.k8s.io/release/$(curl -s https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl"
chmod +x kubectl
sudo mv kubectl /usr/local/bin
```

### 3. AWS CLI: To connect your machine to AWS
- Configure with `aws configure`

---

## 🚀 Create EKS Cluster Using eksctl
```bash
eksctl create cluster \
  --name devops-eks-cluster \
  --version 1.28 \
  --region ap-south-1 \
  --nodegroup-name devops-nodes \
  --node-type t3.medium \
  --nodes 2 \
  --nodes-min 1 \
  --nodes-max 3 \
  --managed
```

This command:
- Creates the control plane (fully managed by AWS)
- Creates a node group of 2 EC2 worker nodes
- Links the worker nodes to the control plane

---

## 🔗 Connect `kubectl` to EKS Cluster
```bash
aws eks update-kubeconfig --region ap-south-1 --name devops-eks-cluster
```
This writes access details to `~/.kube/config`, allowing you to use `kubectl` to talk to your EKS cluster.

---

## ⚙️ Deploy a Sample App
```bash
kubectl create deployment my-app --image=node:18-alpine
```

Expose with a LoadBalancer:
```bash
kubectl expose deployment my-app --type=LoadBalancer --port=80 --target-port=3000
```

---

## 📈 Scaling Deployments
### Manual Scaling:
```bash
kubectl scale deployment my-app --replicas=5
```
- Scheduler assigns pods to available nodes with resources.
- Kubelet on those nodes pulls the image and runs the pod.

### Auto Scaling (HPA):
```yaml
apiVersion: autoscaling/v2
kind: HorizontalPodAutoscaler
metadata:
  name: my-app-hpa
spec:
  scaleTargetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: my-app
  minReplicas: 2
  maxReplicas: 10
  metrics:
    - type: Resource
      resource:
        name: cpu
        target:
          type: Utilization
          averageUtilization: 60
```

---

## 🔍 How Traffic Works in Kubernetes
- Service type `LoadBalancer` creates an AWS ELB
- ELB forwards traffic to the K8s service
- The service load-balances to pods running across **all worker nodes**

---

## 🤖 Control Plane Is Always in Charge
- You only interact with **kubectl**
- kubectl talks to the **control plane**
- Control plane tells worker nodes what to do

You never need to SSH into worker nodes to control pods.

---

## 🎯 Summary
- `eksctl` creates control plane + worker nodes
- `kubectl` communicates with the control plane
- AWS manages the control plane (scales, replicates it internally)
- You manage apps, services, scaling, and monitoring using `kubectl`

---



