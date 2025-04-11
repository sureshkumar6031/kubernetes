# Kubernetes Pod

A **Pod** is the smallest and simplest Kubernetes object. It represents a single instance of a running process in your cluster and encapsulates one or more containers.

## 🔑 Key Characteristics

- **Single IP Address**: Each Pod is assigned a unique IP address within the cluster, allowing communication between containers within the Pod using `localhost`.
- **Shared Storage and Network**: Containers within a Pod share storage volumes and network resources.
- **Lifecycle Management**: Kubernetes manages the lifecycle of Pods, ensuring they are running as expected and restarting them if they fail.

## 🧩 Components

- **Containers**: One or more containers (typically Docker containers) that are tightly coupled and need to share resources.
- **Volume**: Shared storage accessible by all containers within the Pod.
- **Networking**: All containers within a Pod share the same network namespace, including IP address and port space.

## 📦 Use Cases

- Running a single containerized application.
- Running multiple containers that need to work together, such as a web server and a helper container.

---

## 🚀 Example: Creating an Nginx Pod

Below is an example YAML manifest to create a Pod running a single container with the Nginx web server:

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: nginx-pod
  labels:
    app: nginx
spec:
  containers:
  - name: nginx-container
    image: nginx:latest
    ports:
    - containerPort: 80
```

### ✅ Apply the Pod Manifest

You can apply the manifest using `kubectl`:

```bash
kubectl apply -f nginx-pod.yaml
```

### 📌 Verify the Pod

```bash
kubectl get pods
kubectl describe pod nginx-pod
kubectl logs nginx-pod
```

