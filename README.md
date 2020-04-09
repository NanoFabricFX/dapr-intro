# Dapr (Distributed Application Runtime)

## Why Dapr?

### Microservices Architecture

- Loosely coupled for autonomy and idependent scaling
- State and stefull functions and actors
- Service discovery
- Polyglot - each service may be implemented in the most appropriate language, framework, runtime
- Distributed Tracing

### Dapr Features

- Any language, any framework, anywhere
- Microservice building blocks for cloud and edge
- Sidecar architecture
- Developer language SDKs and frameworks (or just use REST)
- Running Dapr on a local developer machine in self hosted mode
- Running Dapr in Kubernetes mode

### Building Blocks

|Building  Block|Description|
|---------------|-----------|
| Service Invocation | Resilient service-to-service invocation enables method calls, including retries, on remote services wherever they are located in the supported hosting environment.|
|State Management|With state management for storing key/value pairs, long running, highly available, stateful services can be easily written alongside stateless services in your application. The state store is pluggable and can include Azure CosmosDB, AWS DynamoDB or Redis among others.|
|Publish and Subscribe Messaging|Publishing events and subscribing to topics|
|Resource Bindings|Resource bindings with triggers builds further on event-driven|
|Actors|A pattern for stateful and stateless objects that make concurrency simple with method and state encapsulation. Dapr provides many capabilities in its actor runtime including concurrency, state, life-cycle management for actor activation/deactivation and timers and reminders to wake-up actors.|
|Distributed Tracing|Dapr supports distributed tracing to easily diagnose.|

### Sidecard Pattern

Co-locate a cohesive set of tasks with the primary application, but place them inside their own process or container, providing a homogeneous interface for platform services across languages.

<div align="center">
<image src="sidecard.png" style="width:90%">
</div>
  
> Note: More inforamtion at: https://docs.microsoft.com/en-us/azure/architecture/patterns/sidecar

## Installation

### Install Dapr CLI

```bash
# Linux
wget -q https://raw.githubusercontent.com/dapr/cli/master/install/install.sh -O - | /bin/bash

sudo dapr init

## There are installers for Windows and MacOS
```

### After Initialization

- Dapr containers are added to docker

## Deployment to Kubernetes

### Minikube & Docker

```bash
# Make sure that docker has been installed
# Install and start minikube
minikube start

# or, get an AKS instance and get the credentials
```

### Install Dapr using helm 3 (Recommended in production)

> Note: RBAC is required

```bash
# Add dapr repo to helm
helm repo add dapr https://daprio.azurecr.io/helm/v1/repo

# Update the repo
helm repo update

# Create the namespace for dapr
kubectl create namespace dapr-system

# Install dapr in the k8s namespace
helm install dapr dapr/dapr --namespace dapr-system

# Check the pods in the dapr-system namespace
kubectl get pods -n dapr-system -o wide
```

Pods:

<div align="center">
<image src="dapr-pods.png" style="width:90%">
</div>

### Or, Install Dapr using CLI

> Note: I had issued with this one, so I used helm

```bash
dapr install --kubernetes
```

## Review the Dapr Samples

### 1.Hello-world (Docker)

```bash
dapr run --app-id nodeapp --app-port 3000 --port 3500 node app.js

dapr run --app-id pythonapp python3 app.py
```
### How does it work

- The node app gets/saves the state using the Dapr state store
- The pythong posts a new value every second app via service invokation to the node app and not directly to the service. The nodeapp saves the new value in the Dap state store.
- Good for development. There's a VS code extension debug into the code.

### 2.Hello-Kubernetes

```bash
cd samples\2.Hello-Kubernetes\deploy

# Install Redis
helm install redis bitnami/redis

# Get the password
REDIS_CACHE=$(kubectl get secret --namespace default redis -o jsonpath="{.data.redis-password}" | base64 --decode)
echo $REDIS_CACHE

# Add password to redis.yaml
gedit redis.yaml &

# Deploy all containers and services
kubectl apply -f .

# See the logs from the node app
kubectl logs --selector=app=node -c node

# Or, Forward the service
kubectl port-forward svc/nodeapp 8080:80

# Open http://localhost:8080/order
```

#### How does it work?

- the redis.yaml creates the dapr state store using redis. Other options include Cosmos DB, SQL, etc.
```bash
apiVersion: dapr.io/v1alpha1
kind: Component
metadata:
  name: statestore
spec:
  type: state.redis
  metadata:
  - name: redisHost
    value: redis-master:6379
  - name: redisPassword
    value: <--REDIS PASSWORD-->
```
- The Dapr sidecars are addid via YAML annotations in the different services
- Python application, using service invocation, makes a POST requests to the node application using the dapr sidecar. It updates the order id every second.
- During the a POST request, the Node application saves the state to the Redis cache. Making a GET request to /order gets the latest order from Redis.
  - > Note: There's no additional code needed to read/write to/from the Redis cache other than to get/post using the sidecard.


### 3.Distributed-calculator

```bash
cd samples\3.Distributed-Calculator\deploy

# Install Redis
helm install redis bitnami/redis

# Get the password
REDIS_CACHE=$(kubectl get secret --namespace default redis -o jsonpath="{.data.redis-password}" | base64 --decode)
echo $REDIS_CACHE

# Add password to redis.yaml
gedit redis.yaml &

# Deploy all containers and services
kubectl apply -f .

# See the logs from the node app
kubectl port-forward svc/calculator-front-end 8080:80

# open http://localhost:8080
```
