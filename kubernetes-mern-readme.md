# Kubernetes MERN Application Deployment

This repository contains Kubernetes configuration files to deploy a MongoDB and Mongo Express web application.

## What is Kubernetes (K8s)?

Kubernetes (K8s) is an open-source container orchestration system for automating deployment, scaling, and management of containerized applications. It groups containers that make up an application into logical units for easy management and discovery.

Key features:
- **Container orchestration**: Automates the deployment and management of containerized applications
- **Self-healing**: Restarts containers that fail, replaces and reschedules containers when nodes die
- **Horizontal scaling**: Scale applications up and down with a simple command or automatically based on CPU usage
- **Service discovery and load balancing**: Kubernetes gives containers their own IP addresses and a single DNS name for a set of containers and can load-balance across them
- **Automated rollouts and rollbacks**: You can describe the desired state for your deployed containers, and Kubernetes can change the actual state to the desired state at a controlled rate
- **Secret and configuration management**: Deploy and update secrets and application configuration without rebuilding your image

## Kubernetes Components Used

### ConfigMap
ConfigMap is a Kubernetes resource that stores non-confidential data in key-value pairs. It allows you to decouple configuration artifacts from image content to keep containerized applications portable.

In our application, we use ConfigMap to store database connection information.

### Secret
Secrets are Kubernetes objects used to store sensitive information such as passwords, OAuth tokens, and SSH keys. Using Secrets means you don't need to include confidential data in your application code or container images.

In our application, we use Secrets to store MongoDB username and password.

### Deployment
A Deployment provides declarative updates for Pods and ReplicaSets. You describe a desired state in a Deployment, and the Deployment Controller changes the actual state to the desired state at a controlled rate.

In our application, we have two deployments:
1. MongoDB deployment
2. Mongo Express web application deployment

### Service
A Service is an abstract way to expose an application running on a set of Pods as a network service. With Kubernetes, you don't need to modify your application to use an unfamiliar service discovery mechanism.

In our application, we have two services:
1. MongoDB service (ClusterIP)
2. Mongo Express web UI service (NodePort)

### Pod
A Pod is the smallest and simplest Kubernetes object. It represents a single instance of a running process in your cluster. Pods contain one or more containers, such as Docker containers.

## Tools Used

### kubectl
kubectl is the Kubernetes command-line tool that allows you to run commands against Kubernetes clusters. You can use kubectl to deploy applications, inspect and manage cluster resources, and view logs.

### Minikube
Minikube is a tool that makes it easy to run Kubernetes locally. Minikube runs a single-node Kubernetes cluster inside a Virtual Machine (VM) on your laptop or desktop for users looking to try out Kubernetes or develop with it day-to-day.

## Common kubectl Commands

### Cluster Information
```bash
# Get cluster information
kubectl cluster-info

# Get nodes in the cluster
kubectl get nodes

# Describe a node
kubectl describe node <node-name>
```

### Working with Resources
```bash
# Get all resources in the current namespace
kubectl get all

# Get all pods
kubectl get pods
kubectl get pod

# Get detailed information about pods
kubectl get pods -o wide

# Get deployments
kubectl get deployments
kubectl get deploy

# Get services
kubectl get services
kubectl get svc

# Get ConfigMaps
kubectl get configmaps
kubectl get cm

# Get Secrets
kubectl get secrets

# Describe resources (detailed information)
kubectl describe pod <pod-name>
kubectl describe deployment <deployment-name>
kubectl describe service <service-name>
kubectl describe configmap <configmap-name>
kubectl describe secret <secret-name>
```

### Logs and Debugging
```bash
# Get logs from a pod
kubectl logs <pod-name>

# Stream logs from a pod
kubectl logs -f <pod-name>

# Execute a command in a pod
kubectl exec -it <pod-name> -- <command>

# Get a shell to a running container
kubectl exec -it <pod-name> -- /bin/bash
```

### Creating and Applying Resources
```bash
# Apply configuration from files
kubectl apply -f <filename>.yaml

# Create resources
kubectl create -f <filename>.yaml
```

### Deleting Resources
```bash
# Delete a specific resource
kubectl delete pod <pod-name>
kubectl delete deployment <deployment-name>
kubectl delete service <service-name>

# Delete all pods
kubectl delete pods --all

# Delete all deployments
kubectl delete deployments --all
kubectl delete deployment --all

# Delete all services
kubectl delete services --all
```

### Minikube Specific Commands
```bash
# Start Minikube
minikube start

# Stop Minikube
minikube stop

# Get Minikube IP
minikube ip

# Open Minikube dashboard
minikube dashboard

# SSH into Minikube VM
minikube ssh
```

## Deployment Steps

1. Start Minikube
   ```bash
   minikube start
   ```

2. Apply the Secret
   ```bash
   kubectl apply -f secret.yaml
   ```

3. Apply the ConfigMap
   ```bash
   kubectl apply -f mongo-config.yaml
   ```

4. Deploy MongoDB
   ```bash
   kubectl apply -f mongo-app.yaml
   ```

5. Deploy the Web Application
   ```bash
   kubectl apply -f web-app.yaml
   ```

6. Verify deployments and services
   ```bash
   kubectl get all
   ```

7. Access the Mongo Express web UI
   ```bash
   # Get Minikube IP
   minikube ip
   
   # Open in browser: <minikube-ip>:30100
   ```

## Architecture Diagram

```
┌─────────────────────────────────────────────────────┐
│                   Kubernetes Cluster                 │
│                                                     │
│  ┌─────────────────┐         ┌────────────────────┐ │
│  │                 │         │                    │ │
│  │ MongoDB Pod     │         │ MongoDB Express Pod│ │
│  │                 │         │                    │ │
│  │ ┌─────────────┐ │         │ ┌────────────────┐ │ │
│  │ │ MongoDB     │ │         │ │ Mongo Express  │ │ │
│  │ │ Container   │ │         │ │ Container      │ │ │
│  │ │             │ │         │ │                │ │ │
│  │ │ Port: 27017 │ │         │ │ Port: 8081     │ │ │
│  │ └─────────────┘ │         │ └────────────────┘ │ │
│  │                 │         │                    │ │
│  └────────┬────────┘         └──────────┬─────────┘ │
│           │                             │           │
│  ┌────────▼────────┐         ┌──────────▼─────────┐ │
│  │                 │         │                    │ │
│  │ MongoDB Service │         │ WebApp Service     │ │
│  │ (ClusterIP)     │         │ (NodePort)         │ │
│  │                 │         │                    │ │
│  │ Port: 27017     │         │ Port: 8081         │ │
│  │                 │         │ NodePort: 30100    │ │
│  └────────┬────────┘         └──────────┬─────────┘ │
│           │                             │           │
└───────────┼─────────────────────────────┼───────────┘
            │                             │
            │                             │
┌───────────▼─────────────┐   ┌───────────▼───────────┐
│                         │   │                       │
│ Internal Access         │   │ External Access       │
│ (Other pods using       │   │ (Browser access)      │
│ mongo-service name)     │   │ <minikube-ip>:30100   │
│                         │   │                       │
└─────────────────────────┘   └───────────────────────┘
```

## Environment Variables

### MongoDB Container
- `MONGO_INITDB_ROOT_USERNAME`: MongoDB username (from Secret)
- `MONGO_INITDB_ROOT_PASSWORD`: MongoDB password (from Secret)

### Mongo Express Container
- `ME_CONFIG_MONGODB_ADMINUSERNAME`: MongoDB admin username (from Secret)
- `ME_CONFIG_MONGODB_ADMINPASSWORD`: MongoDB admin password (from Secret)
- `ME_CONFIG_MONGODB_SERVER`: MongoDB service name (from ConfigMap)

## Notes

- The MongoDB username and password are base64 encoded in the Secret
- The Mongo Express web UI is exposed through NodePort service on port 30100
- The MongoDB service is only accessible within the cluster
- ConfigMap is used to store the MongoDB service name
