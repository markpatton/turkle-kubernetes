# Introduction

These are instructions on how to run Turkle using Kubernetes. The Kubernetes setup should work in production and for development.

# Setup kubernetes

Install kubernetes and some software which can run a kubernetes cluster such as minikube or kind.

Install kind:
```
curl -Lo ./kind https://kind.sigs.k8s.io/dl/v0.8.1/kind-linux-amd64
chmod +x kind 
mv kind ~/local/
```

Create a kubernetes cluster with kind:

```
~/local/kind create cluster
kubectl cluster-info --context kind-kind
kubectl get nodes
```

Install mikube:
```
curl -Lo minikube https://storage.googleapis.com/minikube/releases/latest/minikube-linux-amd64
chmod +x minikube
mv minikube ~/local/
```

Create a kubernetes cluster with minikube:
```
./minikube start
kubectl get nodes
```

# Get Turkle image

Be sure to build the image from Dockerfile-MySQL.

Build image:
```
git clone https://github.com/hltcoe/turkle
cd turkle && docker-compose build turkle
```

For kind, load turkle image into cluster:
```
~/local/kind load docker-image turkle_turkle:latest

docker exec -it kind-control-plane crictl images
```

For minikube, build against its docker registry:
```
eval $(~/local/minikube docker-env)
git clone https://github.com/hltcoe/turkle
cd turkle && docker-compose build turkle
eval $(~/local/minikube docker-env -u)
```

# Deploy turkle

Create a persistent volume claim for MySQL.

```
kubectl create -f mysql-pv-claim.yaml
```

First define a secret for MySQL credentials. Secrets can be created a number of different way. See for example the encrypted
secrets in mysqlsecretsealed.yaml which requires secrets-controller.yaml.

Simple secret creation:
```
kubectl create secret generic mysql --from-literal='rootpassword=badpass' --from-literal='userpassword=moooooo'
```

Deploy MySQL and create a service which exposes it. Check logs and wait for it to start before proceeding.

```
kubectl create -f mysql-deployment.yaml -f mysql-service.yaml
kubectl describe -f mysql-deployment.yaml
kubectl logs mysql-deployment-6bb5fbb5d6-qwmdv
```

Second deploy turkle itself, it should become accessible on port 30000 on the turkle-service node. You may have to modify your routing in able to access it.

```
kubectl create -f turkle-deployment.yaml -f turkle-service.yaml
```

If you are using minikub, access the service with:
```
~/local/minikube service turkle-service
```

You also could port forward for access as http://localhost:8080.

```
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
