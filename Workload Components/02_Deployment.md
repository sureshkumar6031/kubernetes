# Kubernetes Deployment Overview

A **Deployment** is a higher-level Kubernetes object that provides declarative updates to Pods and ReplicaSets. It manages the deployment of application workloads and ensures the desired state of the system matches the actual state.

---

## 🔑 Key Characteristics

- **Declarative Updates**: Define the desired state of your application (e.g., the number of replicas, container images, etc.) and Kubernetes ensures this state is maintained.
- **Rolling Updates**: Automatically updates Pods with new versions while maintaining availability.
- **Rollback**: Easily revert to previous versions in case of failures.
- **Scaling**: Adjust the number of replicas up or down to handle varying loads.

---

## 🧩 Components

- **ReplicaSet**: Ensures a specified number of Pod replicas are running at any given time.
- **Pod Template**: Specifies the configuration for the Pods, including container images, volumes, and labels.

---

## 📦 Use Cases

- Deploying stateless applications.
- Managing the lifecycle of applications with zero downtime during updates.
- Scaling applications to meet demand.
```

---

### 📄 `deployment.yaml` (Manifest file with 3 replicas)

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: sample-app-deployment
  labels:
    app: sample-app
spec:
  replicas: 3
  selector:
    matchLabels:
      app: sample-app
  template:
    metadata:
      labels:
        app: sample-app
    spec:
      containers:
      - name: sample-container
        image: nginx:latest
        ports:
        - containerPort: 80
```

