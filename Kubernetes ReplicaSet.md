# Kubernetes ReplicaSet

A **ReplicaSet** is a Kubernetes object that ensures a specified number of Pod replicas are running at all times. It maintains the desired number of identical Pods and automatically replaces any that fail or are deleted.

---

## 🔑 Key Characteristics

- **Replica Management**: Ensures the desired number of Pod replicas are running.
- **Pod Matching**: Uses **labels** to identify the Pods it manages.
- **Self-Healing**: Automatically replaces failed or deleted Pods to maintain the desired state.

---

## 🧱 Components of ReplicaSet

- **Selector**: A label query used to identify which Pods are managed by the ReplicaSet.
- **Replicas**: Specifies the number of desired Pod replicas.
- **Pod Template**: A template that defines the configuration (image, ports, etc.) for the Pods to be created.

---

## 📦 Use Cases

- Maintaining **high availability** by running multiple instances of a Pod.
- **Auto-replacement** of failed or terminated Pods to maintain application uptime.

---

## 📘 Example: NGINX ReplicaSet YAML

```yaml
apiVersion: apps/v1
kind: ReplicaSet
metadata:
  name: nginx-replicaset
  labels:
    app: nginx
spec:
  replicas: 3
  selector:
    matchLabels:
      app: nginx
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
      - name: nginx-container
        image: nginx:latest
        ports:
        - containerPort: 80
```

---

## 🧾 Explanation of Each Line

| Line | Description |
|------|-------------|
| `apiVersion: apps/v1` | Defines the API version used to create the ReplicaSet. |
| `kind: ReplicaSet` | Specifies that the object type is a ReplicaSet. |
| `metadata:` | Metadata about the ReplicaSet object. |
| `name: nginx-replicaset` | The name of the ReplicaSet. |
| `labels:` | Labels attached to this ReplicaSet object (not to be confused with the Pod labels). |
| `app: nginx` | A key-value label pair to help identify the app. |
| `spec:` | Specifies the desired state and behavior of the ReplicaSet. |
| `replicas: 3` | The number of Pod replicas to be maintained at all times. |
| `selector:` | Defines how the ReplicaSet finds which Pods to manage. |
| `matchLabels:` | The ReplicaSet matches Pods with this label. |
| `app: nginx` | Must match the labels defined inside the Pod template. |
| `template:` | A Pod template used to create new Pods. |
| `metadata:` | Metadata for the Pods that will be created. |
| `labels:` | Labels that the Pods will carry; must match the selector above. |
| `spec:` | Defines what runs inside the Pods. |
| `containers:` | List of containers inside the Pod. |
| `- name: nginx-container` | The name of the container. |
| `image: nginx:latest` | Docker image used for the container (latest version of NGINX). |
| `ports:` | Exposes ports for networking. |
| `- containerPort: 80` | Exposes port 80 inside the container (default for NGINX). |

---

## ✅ Summary

A ReplicaSet ensures availability, scalability, and self-healing of Pods in Kubernetes. It's commonly used to keep multiple instances of stateless applications like **NGINX** running and resilient to failures.

---

```

