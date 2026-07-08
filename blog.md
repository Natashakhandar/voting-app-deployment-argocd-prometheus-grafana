# Deploying a Scalable Voting App on Kubernetes with GitOps & Monitoring

In today's cloud-native ecosystem, Kubernetes is the gold standard for container orchestration. But deploying an application is only half the battle—you also need robust continuous deployment (GitOps) and deep cluster monitoring.

In this blog, I will walk you through a complete, production-grade deployment of a microservices-based Voting Application on a local Kubernetes cluster using **Kind**. We will also integrate **ArgoCD** for automated deployments and the **Kube-Prometheus-Stack** (Prometheus & Grafana) to monitor our cluster’s health.

---

### Step 1: Creating a Kubernetes Cluster with Kind

First, let's create a 3-node Kubernetes cluster using Kind with a configuration file:
```bash
kind create cluster --config=config.yml
```

Verify that the cluster and nodes are running:
```bash
kubectl cluster-info --context kind-kind
kubectl get nodes
```

---

### Step 2: Managing Kubernetes Pods

Check the Docker containers running (these represent our Kind nodes):
```bash
docker ps
```

List all Kubernetes pods across all namespaces to ensure core components are running:
```bash
kubectl get pods -A
```

---

### Step 3: Cloning and Deploying the Voting App

Instead of deploying everything manually, we will apply the specifications for our microservices (DB, Redis, Voting App, Result App, and Worker).

Clone the repository and apply the configurations:
```bash
git clone https://github.com/Natashakhandar/voting-app-deployment-argocd-prometheus-grafana.git
cd voting-app-deployment-argocd-prometheus-grafana/
kubectl apply -f k8s-specifications/
```

Verify that the application pods are running:
```bash
kubectl get pods
```
![Terminal Get Pods](https://raw.githubusercontent.com/Natashakhandar/voting-app-deployment-argocd-prometheus-grafana/main/terminal-get-pods.jpeg)

Forward local ports so you can access the Voting and Result web interfaces:
```bash
kubectl port-forward service/vote 5000:5000 --address=0.0.0.0 &
kubectl port-forward service/result 5001:5001 --address=0.0.0.0 &
```

You can now view the app UI on your local browser:
![Voting App UI](https://raw.githubusercontent.com/Natashakhandar/voting-app-deployment-argocd-prometheus-grafana/main/voting-app-ui.jpeg)

---

### Step 4: Installing Argo CD (GitOps)

To automate future deployments, we will set up Argo CD.

Create a namespace and apply the official Argo CD installation manifest:
```bash
kubectl create namespace argocd
kubectl apply -n argocd -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml
```

Expose the Argo CD server and port-forward it to access the UI:
```bash
kubectl patch svc argocd-server -n argocd -p '{"spec": {"type": "NodePort"}}'
kubectl port-forward -n argocd service/argocd-server 8443:443 &
```

Retrieve the initial admin password:
```bash
kubectl get secret -n argocd argocd-initial-admin-secret -o jsonpath="{.data.password}" | base64 -d && echo
```

![ArgoCD UI](https://raw.githubusercontent.com/Natashakhandar/voting-app-deployment-argocd-prometheus-grafana/main/argocd-ui.jpeg)

---

### Step 5: Installing the Kubernetes Dashboard

Deploy the Kubernetes Dashboard for a visual representation of your cluster:
```bash
kubectl apply -f https://raw.githubusercontent.com/kubernetes/dashboard/v2.7.0/aio/deploy/recommended.yaml
```

Create an admin-user token to authenticate:
```bash
kubectl -n kubernetes-dashboard create token admin-user
```
![K8s Dashboard Status](https://raw.githubusercontent.com/Natashakhandar/voting-app-deployment-argocd-prometheus-grafana/main/k8s-dashboard-status.jpeg)
![K8s Dashboard Workloads](https://raw.githubusercontent.com/Natashakhandar/voting-app-deployment-argocd-prometheus-grafana/main/k8s-dashboard-workloads.jpeg)

---

### Step 6: Install Kube Prometheus Stack (Monitoring & Grafana)

Add the Helm repositories and install the Prometheus stack:
```bash
helm repo add prometheus-community https://prometheus-community.github.io/helm-charts
helm repo update
kubectl create namespace monitoring
helm install kind-prometheus prometheus-community/kube-prometheus-stack --namespace monitoring --set prometheus.service.nodePort=30000 --set prometheus.service.type=NodePort --set grafana.service.nodePort=31000 --set grafana.service.type=NodePort
```

Check Prometheus targets to ensure metrics are being scraped:
![Prometheus Targets](https://raw.githubusercontent.com/Natashakhandar/voting-app-deployment-argocd-prometheus-grafana/main/prom-targets.jpeg)

Port-forward Prometheus and Grafana:
```bash
kubectl port-forward svc/kind-prometheus-kube-prome-prometheus -n monitoring 9090:9090 --address=0.0.0.0 &
kubectl port-forward svc/kind-prometheus-grafana -n monitoring 31000:80 --address=0.0.0.0 &
```

With Grafana running, you can create and view comprehensive Kubernetes dashboards:
![Grafana K8s Dashboard Overview](https://raw.githubusercontent.com/Natashakhandar/voting-app-deployment-argocd-prometheus-grafana/main/grafana-k8s-dash-1.jpeg)
![Grafana K8s Dashboard Details](https://raw.githubusercontent.com/Natashakhandar/voting-app-deployment-argocd-prometheus-grafana/main/grafana-k8s-dash-2.jpeg)
![Grafana Bar Chart Editing](https://raw.githubusercontent.com/Natashakhandar/voting-app-deployment-argocd-prometheus-grafana/main/grafana-bar.jpeg)
![Grafana Line Chart Editing](https://raw.githubusercontent.com/Natashakhandar/voting-app-deployment-argocd-prometheus-grafana/main/grafana-line.jpeg)

---

### Step 7: Useful Prometheus Queries

Use these PromQL queries to monitor your cluster's health:

**CPU Usage Percentage (Default Namespace):**
```bash
sum (rate (container_cpu_usage_seconds_total{namespace="default"}[1m])) / sum (machine_cpu_cores) * 100
```
![Prometheus CPU Graph](https://raw.githubusercontent.com/Natashakhandar/voting-app-deployment-argocd-prometheus-grafana/main/prom-cpu.jpeg)

**Memory Usage by Pod:**
```bash
sum (container_memory_usage_bytes{namespace="default"}) by (pod)
```
![Prometheus Memory Graph](https://raw.githubusercontent.com/Natashakhandar/voting-app-deployment-argocd-prometheus-grafana/main/prom-memory.jpeg)

**Network Traffic by Pod:**
```bash
sum(rate(container_network_receive_bytes_total{namespace="default"}[5m])) by (pod)
sum(rate(container_network_transmit_bytes_total{namespace="default"}[5m])) by (pod)
```
![Prometheus Network Receive](https://raw.githubusercontent.com/Natashakhandar/voting-app-deployment-argocd-prometheus-grafana/main/prom-network-rx.jpeg)
![Prometheus Network Transmit](https://raw.githubusercontent.com/Natashakhandar/voting-app-deployment-argocd-prometheus-grafana/main/prom-network-tx.jpeg)

---

### Conclusion

Deploying applications locally with Kind provides a lightweight, highly efficient way to test your Kubernetes manifests. By integrating tools like ArgoCD and Prometheus, you ensure your setup is production-ready right from the start!

You can find the entire source code and configuration files for this project on my GitHub:
[https://github.com/Natashakhandar/voting-app-deployment-argocd-prometheus-grafana](https://github.com/Natashakhandar/voting-app-deployment-argocd-prometheus-grafana)

Happy coding!
