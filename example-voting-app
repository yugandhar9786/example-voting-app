#!/bin/bash

# ================================================
# 🚀 Kubernetes Resource Deployment Script
# ================================================

# Exit immediately if a command exits with a non-zero status
set -e

# Directory where YAML files are located
K8S_DIR="k8s-specifications"

echo "------------------------------------------"
echo "🚀 Starting Kubernetes deployment process..."
echo "------------------------------------------"

# Check if kubectl is installed
if ! command -v kubectl &> /dev/null; then
    echo "❌ kubectl not found! Please install kubectl before running this script."
    exit 1
fi

# Check if kubeconfig context is set
if ! kubectl cluster-info &> /dev/null; then
    echo "❌ Kubernetes cluster not reachable. Please configure your kubeconfig."
    exit 1
fi

# Apply all YAML files in order (to respect dependencies)
echo "✅ Deploying database resources..."
kubectl apply -f $K8S_DIR/db-deployment.yaml
kubectl apply -f $K8S_DIR/db-service.yaml

echo "✅ Deploying Redis resources..."
kubectl apply -f $K8S_DIR/redis-deployment.yaml
kubectl apply -f $K8S_DIR/redis-service.yaml

echo "✅ Deploying Worker..."
kubectl apply -f $K8S_DIR/worker-deployment.yaml

echo "✅ Deploying Vote application..."
kubectl apply -f $K8S_DIR/vote-deployment.yaml
kubectl apply -f $K8S_DIR/vote-service.yaml

echo "✅ Deploying Result application..."
kubectl apply -f $K8S_DIR/result-deployment.yaml
kubectl apply -f $K8S_DIR/result-service.yaml

echo "✅ Deploying Ingress..."
kubectl apply -f $K8S_DIR/ingress.yaml

# If you have an extra deployment manifest
if [ -f "$K8S_DIR/deploy.yaml" ]; then
    echo "✅ Applying additional deployment file..."
    kubectl apply -f $K8S_DIR/deploy.yaml
fi

echo "------------------------------------------"
echo "🎉 All Kubernetes resources deployed successfully!"
echo "------------------------------------------"

# Show all running pods and services for verification
echo "📦 Current Pods:"
kubectl get pods -o wide

echo "🧩 Current Services:"
kubectl get svc -o wide

echo "🌍 Current Ingress:"
kubectl get ingress -o wide

