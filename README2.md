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
✓ AmazonEKSClusterPolicy  
✓ AmazonEKSServicePolicy  
✓ AmazonEC2FullAccess
✓ EKSProvisioningMinimalPolicy
✓ IAMFullAccess (optional during testing)
 ```
Customize Policy (EKSProvisioningMinimalPolicy):
Go to IAM → Policies → Create policy → JSON tab, and paste this:
```bash
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Effect": "Allow",
            "Action": [
                "eks:*",
                "ec2:Describe*",
                "ec2:CreateSecurityGroup",
                "ec2:CreateTags",
                "ec2:AuthorizeSecurityGroupIngress",
                "ec2:AuthorizeSecurityGroupEgress",
                "ec2:RevokeSecurityGroupEgress",
                "ec2:DeleteSecurityGroup",
                "ec2:DeleteTags",
                "iam:CreateRole",
                "iam:DeleteRole",
                "iam:AttachRolePolicy",
                "iam:PutRolePolicy",
                "iam:GetRole",
                "iam:PassRole",
                "iam:ListRoles",
                "cloudformation:*",
                "autoscaling:*",
                "logs:*"
            ],
            "Resource": "*"
        }
    ]
}
```             

Enter:
- Access Key ID
- Secret Access Key
- Default region (e.g., ap-south-1)
- Output format (e.g., json)


### Install `eksctl`

```bash
curl -s --location "https://github.com/eksctl-io/eksctl/releases/latest/download/eksctl_$(uname -s)_amd64.tar.gz" | tar xz -C /tmp
sudo mv /tmp/eksctl /usr/local/bin/
```

### Install `kubectl`

```bash
curl -LO https://dl.k8s.io/release/v1.29.1/bin/linux/amd64/kubectl
chmod +x kubectl
sudo mv kubectl /usr/local/bin/
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

