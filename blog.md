# Deploying a Scalable Voting Application on a Local Kubernetes Cluster using Kind

In today's cloud-native ecosystem, Kubernetes is the standard for container orchestration. However, setting up a full-blown Kubernetes cluster for local development and testing can be resource-intensive. This is where **Kind** (Kubernetes IN Docker) comes in. Kind allows you to run local Kubernetes clusters using Docker container "nodes".

In this guide, I will walk you through the process of deploying a microservices-based Voting Application on a local Kubernetes cluster using Kind.

## Prerequisites

Before we begin, ensure you have the following installed on your local machine:
- Docker
- Git

## Step 1: Creating a Kubernetes Cluster with Kind

First, we need to create a Kubernetes cluster. We will create a multi-node cluster (one control-plane and two worker nodes) using a configuration file.

Create a `config.yml` file with your cluster configuration and run the following command:

```bash
kind create cluster --config=config.yml
```

![Cluster Creation](./01-cluster-creation.jpeg)
![Cluster Configuration](./02-cluster-config.jpeg)

## Step 2: Verifying the Cluster

Once the cluster is created, we can verify its status and the nodes that were provisioned.

```bash
kubectl cluster-info --context kind-kind
kubectl get nodes
kind get clusters
```

![Cluster Info](./03-cluster-info.jpeg)
![Get Nodes](./04-get-nodes.jpeg)

## Step 3: Installing kubectl (If not already installed)

If you don't have `kubectl` installed, you can download and install it using the following commands:

```bash
curl -o kubectl https://amazon-eks.s3.us-west-2.amazonaws.com/1.19.6/2021-01-05/bin/linux/amd64/kubectl
chmod +x ./kubectl
sudo mv ./kubectl /usr/local/bin
kubectl version --short --client
```

![Kubectl Installation](./05-kubectl-install.jpeg)

Let's ensure Docker is running our Kind containers properly:

```bash
docker ps
```

![Docker PS](./06-docker-ps.jpeg)

## Step 4: Cloning the Voting App Repository

We will use the standard Docker Example Voting App. It's a great example of a multi-container application. Let's clone the repository.

```bash
git clone https://github.com/dockersamples/example-voting-app.git
cd example-voting-app/
```

![Git Clone](./07-git-clone.jpeg)
![Navigate Repo](./08-navigate-repo.jpeg)

## Step 5: Deploying the Application

The repository contains Kubernetes specification files (YAML) for Deployments and Services. We will apply all configurations in the `k8s-specifications/` directory.

```bash
kubectl apply -f k8s-specifications/
```

![Apply Specifications](./09-apply-specs.jpeg)
![Resource Creation](./10-resource-creation.jpeg)

## Step 6: Verifying the Deployment

Let's check if all our Pods, Services, and Deployments are up and running.

```bash
kubectl get pods -A
```

![Get Pods](./11-get-pods.jpeg)

To view all resources created in the default namespace:

```bash
kubectl get all
```

![Get All Resources](./12-get-all.jpeg)

## Step 7: Accessing the Application

Since the services are running inside the Kind cluster, we need to port-forward them to our local machine to access the web interfaces.

Forward the voting service (where users cast votes) to port 5000:
```bash
kubectl port-forward service/vote 5000:5000 --address=0.0.0.0 &
```

Forward the result service (where we see the outcome) to port 5001:
```bash
kubectl port-forward service/result 5001:5001 --address=0.0.0.0 &
```

![Port Forwarding Vote](./13-port-forward-vote.jpeg)
![Port Forwarding Result](./14-port-forward-result.jpeg)

## Conclusion

You can now open your browser and navigate to `http://localhost:5000` to cast your vote, and `http://localhost:5001` to view the results. 

![Application UI](./15-app-ui.jpeg)

Deploying applications locally with Kind provides a lightweight, scalable, and highly efficient way to test your Kubernetes manifests before pushing them to production. 

Happy coding!
