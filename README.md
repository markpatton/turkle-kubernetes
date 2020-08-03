# Introduction

These are instructions on how to run Turkle using Kubernetes.

# Setup kubernetes

Install kubernetes and some software which can run a kubernetes cluster such as kind.

Install kind:
```
curl -Lo ./kind https://kind.sigs.k8s.io/dl/v0.8.1/kind-linux-amd64
chmod +x kind 
mv kind ~/local/
```

Create a kubernetes cluster with kind

```
~/local/kind create cluster
kubectl cluster-info --context kind-kind
kubectl get nodes
```

# Get Turkle image

Build image:
```
git clone https://github.com/hltcoe/turkle
cd turkle && docker-compose build turkle
```

Load turkle image into cluster:
```
~/local/kind load docker-image turkle_turkle:latest 

docker exec -it kind-control-plane crictl images
```

# Deploy turkle

First deploy and expose MySQL service. Check logs and wait for it to start before proceeding.

```
kubectl create -f mysql-deployment.yaml
kubectl expose deployment mysql-deployment --type=NodePort --name=mysql-service
```

Second deploy turkle itself. For production you would expose turkle using a load balancer. For development, port forward so Turkle can be accessed as http://localhost:8080.

```
kubectl create -f turkle-deployment.yaml
# kubectl expose deployment turkle-deployment --type=LoadBalancer --name=turkle-service

kubectl port-forward deployment/turkle-deployment 8080:8080
```

# Configure turkle

Set the root password in the turkle container.

```
kubectl get pods
kubectl exec --stdin --tty turkle-deployment-6d76f997d8-vqqzt -- /bin/bash
python3 manage.py createsuperuser
```

# Use turkle

Login with the root username and password.



