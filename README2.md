# ELK Stack Setup on Kubernetes (EKS)

## Overview

This guide provides a clean and clear walkthrough to install and configure the **ELK Stack** (Elasticsearch, Logstash, and Kibana) on a Kubernetes cluster using **AWS EKS**. This includes setting up the admin machine, provisioning the cluster using `eksctl`, and deploying ELK using Helm charts.

---

## Prerequisites

- AWS account with IAM permissions to create EKS cluster and resources
- AWS CLI installed and configured
- `eksctl` installed on admin machine
- `kubectl` installed
- `helm` installed

---

## 1. Set Up Admin Machine

### Install and Configure AWS CLI

Install AWS CLI:
```bash
curl "https://awscli.amazonaws.com/awscli-exe-linux-x86_64.zip" -o "awscliv2.zip"
unzip awscliv2.zip
sudo ./aws/install
```

Configure AWS CLI with your credentials:
```bash
aws configure
```

Set Up IAM Role for EKS (if needed):

Ensure you have the required IAM roles set up for EKS. eksctl can create the necessary roles for 
you when you create a cluster, but the IAM user you use must have permissions like:
 ```bash
 eks:CreateCluster

 eks:DescribeCluster

 eks:UpdateClusterConfig
 ```

Enter:
- Access Key ID
- Secret Access Key
- Default region (e.g., ap-south-1)
- Output format (e.g., json)

### Install `eksctl`

```bash
curl -s --location "https://github.com/eksctl-io/eksctl/releases/latest/download/eksctl_$(uname -s)_amd64.tar.gz" | tar xz -C /tmp
sudo mv /tmp/eksctl /usr/local/bin
```

### Install `kubectl`

```bash
curl -LO "https://dl.k8s.io/release/$(curl -s https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl"
chmod +x kubectl
sudo mv kubectl /usr/local/bin
```

### Install Helm

```bash
curl https://raw.githubusercontent.com/helm/helm/main/scripts/get-helm-3 | bash
```

---

## 2. Create an EKS Cluster

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

This creates a highly available Kubernetes cluster with:

- Managed control plane by AWS (replicated across 3 AZs)
- 2 worker nodes (EC2 instances of type `t3.medium`)

---

## 3. Update `kubeconfig` to Access the Cluster

```bash
aws eks update-kubeconfig --region ap-south-1 --name devops-eks-cluster
```

You can now run:

```bash
kubectl get nodes
```

To verify access to the worker nodes.

---

## 4. Prepare Helm for ELK Deployment

### Add Elastic Helm Repository

```bash
helm repo add elastic https://helm.elastic.co
helm repo update
```

This step tells Helm where to find official charts for Elasticsearch, Logstash, and Kibana.

---

## 5. Deploy ELK Stack

### Deploy Elasticsearch

```bash
helm install elasticsearch elastic/elasticsearch -n kube-system
```

### Deploy Kibana

```bash
helm install kibana elastic/kibana -n kube-system
```

### Deploy Logstash (Optional)

```bash
helm install logstash elastic/logstash -n kube-system
```

---

## Notes on Instance Type and Cost

- `t3.medium` (2 vCPU, 4GB RAM) is recommended for stable performance.
- `t2.small` is not sufficient for ELK stack — not enough CPU or RAM.
- Use **Spot Instances** or reduce nodes to save cost.

---

## Summary

- EKS gives you a managed Kubernetes control plane across 3 AZs.
- Worker nodes are your EC2 instances (e.g., `t3.medium`)
- Helm charts simplify the deployment of ELK components.
- All configuration and access is done from the admin machine.

---

## Next Steps

Once ELK is deployed, you can:

- Forward logs from apps to Logstash
- Monitor logs using Kibana
- Integrate Filebeat or Fluentd for log shipping

Ready for advanced Kubernetes DevOps projects after this setup.

