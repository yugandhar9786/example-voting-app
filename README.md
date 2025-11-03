# Example Voting App

A simple distributed application running across multiple Docker containers.

## Getting started

Download [Docker Desktop](https://www.docker.com/products/docker-desktop) for Mac or Windows. [Docker Compose](https://docs.docker.com/compose) will be automatically installed. On Linux, make sure you have the latest version of [Compose](https://docs.docker.com/compose/install/).

This solution uses Python, Node.js, .NET, with Redis for messaging and Postgres for storage.

Run in this directory to build and run the app:

```shell
docker compose up
```

The `vote` app will be running at [http://localhost:8080](http://localhost:8080), and the `results` will be at [http://localhost:8081](http://localhost:8081).

Alternately, if you want to run it on a [Docker Swarm](https://docs.docker.com/engine/swarm/), first make sure you have a swarm. If you don't, run:

```shell
docker swarm init
```

Once you have your swarm, in this directory run:

```shell
docker stack deploy --compose-file docker-stack.yml vote
```

## Run the app in Kubernetes

The folder k8s-specifications contains the YAML specifications of the Voting App's services.

Run the following command to create the deployments and services. Note it will create these resources in your current namespace (`default` if you haven't changed it.)

```shell
kubectl create -f k8s-specifications/
```

The `vote` web app is then available on port 31000 on each host of the cluster, the `result` web app is available on port 31001.

To remove them, run:

```shell
kubectl delete -f k8s-specifications/
```

## Architecture

![Architecture diagram](architecture.excalidraw.png)

* A front-end web app in [Python](/vote) which lets you vote between two options
* A [Redis](https://hub.docker.com/_/redis/) which collects new votes
* A [.NET](/worker/) worker which consumes votes and stores them in…
* A [Postgres](https://hub.docker.com/_/postgres/) database backed by a Docker volume
* A [Node.js](/result) web app which shows the results of the voting in real time

## Notes

The voting application only accepts one vote per client browser. It does not register additional votes if a vote has already been submitted from a client.

This isn't an example of a properly architected perfectly designed distributed app... it's just a simple
example of the various types of pieces and languages you might see (queues, persistent data, etc), and how to
deal with them in Docker at a basic level.


# 🚀 Setup NGINX Ingress Controller on AWS EKS

This guide provides all the commands and YAML configurations needed to install and configure the **NGINX Ingress Controller** on your **AWS EKS** cluster.

---

## 📋 Prerequisites

Before installation, ensure you have:

- ✅ An existing AWS EKS Cluster (running and healthy)
- ✅ `kubectl` installed and configured
- ✅ `aws` CLI installed and configured
- ✅ `helm` installed (v3 or higher)
- ✅ Worker nodes in **public subnets** or NAT-enabled private subnets

---

## ⚙️ Step 1: Configure kubectl for your EKS Cluster

```bash
aws eks --region <region-name> update-kubeconfig --name <cluster-name>

# Verify connection
kubectl get nodes

✅ You should see your EKS worker nodes listed as Ready.


🔐 Step 2: Enable IAM OIDC Provider

Ingress controllers need IAM permissions to create AWS Load Balancers.
Check if the OIDC provider is already associated:
```bash
aws eks describe-cluster --name <cluster-name> \
  --query "cluster.identity.oidc.issuer" \
  --output text

If not, create it:
```bash
eksctl utils associate-iam-oidc-provider \
  --region <region-name> \
  --cluster <cluster-name> \
  --approve

  🧾 Step 3: Create IAM Policy for LoadBalancer Controller

Download the AWS Load Balancer Controller IAM policy:
```bash
curl -o iam_policy.json https://raw.githubusercontent.com/kubernetes-sigs/aws-load-balancer-controller/v2.7.0/docs/install/iam_policy.json

Create the IAM policy in AWS:
```bash
aws iam create-policy \
  --policy-name AWSLoadBalancerControllerIAMPolicy \
  --policy-document file://iam_policy.json

  👤 Step 4: Create Service Account with IAM Role

Create a Kubernetes service account and attach the IAM policy:
```bash
eksctl create iamserviceaccount \
  --cluster=<cluster-name> \
  --namespace=kube-system \
  --name=aws-load-balancer-controller \
  --attach-policy-arn=arn:aws:iam::<ACCOUNT_ID>:policy/AWSLoadBalancerControllerIAMPolicy \
  --approve

📦 Step 5: Install NGINX Ingress Controller using Helm

Add the NGINX Helm repo and update:
```bash
helm repo add ingress-nginx https://kubernetes.github.io/ingress-nginx
helm repo update

Install the controller in a dedicated namespace:
```bash
helm install ingress-nginx ingress-nginx/ingress-nginx \
  --namespace ingress-nginx \
  --create-namespace \
  --set controller.publishService.enabled=true

🔍 Step 6: Verify Installation

Check if the controller pods are running:
```bash
kubectl get pods -n ingress-nginx

Check if a LoadBalancer service has been created:
```bash
kubectl get svc -n ingress-nginx

🌐 Step 7: Deploy a Test Application

Create a simple NGINX deployment and service:
```bash
kubectl create deployment test-nginx --image=nginx
kubectl expose deployment test-nginx --port=80 --type=ClusterIP



