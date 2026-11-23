# Deploying the Tic Tac Toe App on Kubernetes

I deployed my Node.js Tic Tac Toe (TTT) app on a local Kubernetes cluster in two stages:

1. **App only:** 3 replicas, exposed with a NodePort service
2. **App + database:** the same app, plus a Mongo database (1 replica)

My notes from the code-along are in [k8s-code-along.md](k8s-code-along.md).

---

## Prerequisites

- App image on Docker Hub: `thantzintun234/tech613-tttapp:1.2.0`
- Database image: `mongo:7.0` (the official Mongo image)

---

## Architecture
![alt text](<yaml-definitions/autodraw 11_20_2026.png>)

- The Deployment keeps 3 identical app pods running (via a ReplicaSet).
- The app service lets me open the app from my browser.
- In part 2, the app pods talk to Mongo through an internal (ClusterIP) service. Nothing outside the cluster can reach the database.

---

## Part 1: App only

Folder: `yaml-definitions/local-ttt-app-deploy`

I copied the Nginx files and changed the names, labels, image and ports.

### `ttt-deploy.yml`

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: ttt-app-deployment
spec:
  replicas: 3
  selector:
    matchLabels:
      app: ttt-app
  template:
    metadata:
      labels:
        app: ttt-app
    spec:
      containers:
        - name: ttt-app
          image: thantzintun234/tech613-tttapp:1.2.0
          ports:
            - containerPort: 3000
```

### `ttt-service.yml`

```yaml
apiVersion: v1
kind: Service
metadata:
  name: ttt-service
  namespace: default
spec:
  ports:
  - nodePort: 30001 # range is 30000-32767
    port: 3000
    targetPort: 3000
  selector:
    app: ttt-app  # must match the deployment label
  type: NodePort
```

### Deploy

```bash
kubectl apply -f ttt-deploy.yml
kubectl apply -f ttt-service.yml
kubectl get all
```
---

## Part 2: App + database

Folder: `yaml-definitions/local-ttt-app-db-deploy`

I kept the app files and added two Mongo files.

### `mongo-deploy.yml`

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: mongo-deployment
spec:
  replicas: 1
  selector:
    matchLabels:
      app: mongo
  template:
    metadata:
      labels:
        app: mongo
    spec:
      containers:
        - name: mongo
          image: mongo:7.0
          ports:
            - containerPort: 27017
```

### `mongo-service.yml`

```yaml
apiVersion: v1
kind: Service
metadata:
  name: mongo-service
spec:
  type: ClusterIP   # internal only
  selector:
    app: mongo
  ports:
    - port: 27017
      targetPort: 27017
```

### Change in `ttt-deploy.yml`

I added an environment variable so the app can find the database:

```yaml
          env:
            - name: MONGODB_URI
              value: mongodb://mongo-service:27017/tictactoe
```

In Docker Compose the host was `mongo`. In Kubernetes it is the service name, `mongo-service`.

### Deploy

```bash
kubectl apply -f mongo-deploy.yml
kubectl apply -f mongo-service.yml
kubectl apply -f ttt-deploy.yml
kubectl apply -f ttt-service.yml
kubectl get all
```


## What I learned

- Kubernetes keeps the **desired state** (3 replicas) and replaces pods that fail.
- A **service** gives pods a stable address, and it finds them using **labels**.
- Containers in the cluster reach each other by **service name**, not by IP.
- Use **NodePort** to expose the app, and **ClusterIP** to keep the database private.
- Mongo has no volume here, so its data is lost if the pod is replaced. A persistent volume would fix that.
