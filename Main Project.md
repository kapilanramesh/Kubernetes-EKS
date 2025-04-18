Sure! Here’s a detailed README for your **Prometheus + Grafana Monitoring Stack on EKS** project:

---

# **Prometheus + Grafana Monitoring Stack on Amazon EKS**

## **Project Overview**
This project sets up a Prometheus and Grafana monitoring stack on Amazon EKS (Elastic Kubernetes Service) using Helm. The monitoring stack includes Prometheus for collecting metrics and Grafana for visualizing them. The stack is deployed using the Helm chart `prometheus-community/kube-prometheus-stack`, which also includes Alertmanager and various exporters for Kubernetes metrics.

### **Components**
- **Prometheus**: Open-source monitoring and alerting toolkit designed for reliability and scalability.
- **Grafana**: Open-source visualization and analytics platform for monitoring data collected by Prometheus.
- **Alertmanager**: Manages alerts sent by Prometheus.
- **Node Exporter**: Exposes hardware and OS metrics from the nodes.
- **Kube State Metrics**: Provides information on the state of Kubernetes clusters.

## **Prerequisites**
- AWS account with EKS cluster set up.
- AWS CLI configured.
- Helm 3.x installed.
- kubectl installed and configured to interact with your EKS cluster.
- IAM role with sufficient permissions for deploying resources to EKS.
- Security Group and Network setup to allow inbound connections to the EKS services (Grafana access).

## **Steps to Deploy Prometheus + Grafana Monitoring Stack**

### **1. Install the Prometheus-Community Helm Repository**
First, add the Helm repository for Prometheus community charts:

```bash
helm repo add prometheus-community https://prometheus-community.github.io/helm-charts
helm repo update
```

### **2. Install the Kube Prometheus Stack**
Install the kube-prometheus-stack Helm chart. This will deploy Prometheus, Grafana, Alertmanager, and the required exporters in your EKS cluster:

```bash
helm install my-prometheus prometheus-community/kube-prometheus-stack -n monitoring --create-namespace
```

### **3. Access Grafana**
To access the Grafana dashboard via a **NodePort**, update the Grafana service:

1. **Export the Service to a NodePort**:
   Create a YAML file `prometheus-nodeport.yaml` with the following content:

   ```yaml
   apiVersion: v1
   kind: Service
   metadata:
     name: my-prometheus-grafana
     namespace: monitoring
   spec:
     type: NodePort
     ports:
       - port: 80
         targetPort: 3000
         nodePort: 32000
     selector:
       app.kubernetes.io/name: grafana
   ```

2. **Apply the YAML File**:
   Apply the YAML to update the Grafana service:

   ```bash
   kubectl apply -f prometheus-nodeport.yaml
   ```

### **4. Access Grafana in Your Browser**
After applying the service update, you can access the Grafana UI in your browser using the following URL:

```
http://<EC2_Public_IP>:32000
```

- Replace `<EC2_Public_IP>` with the public IP address of your EC2 instance running the EKS node.

### **5. Login to Grafana**
The default login credentials are:
- **Username**: admin
- **Password**: Fetch the password using the following command:

```bash
kubectl --namespace monitoring get secrets my-prometheus-grafana -o jsonpath="{.data.admin-password}" | base64 -d ; echo
```

### **6. Configure Prometheus as a Data Source in Grafana**
Once logged into Grafana, you can add Prometheus as a data source:
- Go to **Configuration > Data Sources**.
- Add **Prometheus** as a data source, and set the URL to `http://prometheus-operated:9090`.

### **7. Create Dashboards in Grafana**
Grafana will automatically create some dashboards for Prometheus metrics. You can also create custom dashboards to monitor specific metrics.

### **8. Verify Node Exporter is Running**
To ensure node metrics are being collected, check the status of the `prometheus-node-exporter` pods:

```bash
kubectl get pods -n monitoring -l app.kubernetes.io/name=prometheus-node-exporter
```

If the pods are in `Pending` or `CrashLoopBackOff` state, troubleshoot by checking the logs:

```bash
kubectl logs -n monitoring <prometheus-node-exporter-pod-name>
```

### **9. Set Up Alerts in Grafana**
To configure alerts in Grafana, navigate to the **Alerting** section:
- Create alert rules based on Prometheus metrics.
- Set up alert notifications (email, Slack, etc.).

### **10. Clean Up the Resources**
To delete the monitoring stack when you're done, use the following command:

```bash
helm uninstall my-prometheus -n monitoring
kubectl delete namespace monitoring
```

## **Usage Instructions**
- **Grafana**: Visit Grafana at `http://<EC2_Public_IP>:32000` and use the default credentials to log in. You can customize dashboards, set alerts, and explore metrics from Prometheus.
- **Prometheus**: Prometheus is running in the background, scraping metrics from Kubernetes and other services. You can access the Prometheus UI by setting up a NodePort or using port-forwarding.

## **Troubleshooting**
- **Port-Forwarding Issue**: If you can’t access Grafana, make sure the NodePort is open in your EC2 security group and the Kubernetes service is properly configured.
- **Node Exporter Not Running**: Ensure that the `prometheus-node-exporter` pods are running and check their logs for any errors.
- **Alertmanager Configuration**: If you have configured Alertmanager, ensure that the email or Slack integration is working properly by checking the Alertmanager logs.

## **Conclusion**
This setup will allow you to monitor your Kubernetes cluster’s performance with Prometheus and visualize the data in Grafana. You can further customize this setup to include additional exporters or modify alerting rules as per your requirements.

---
