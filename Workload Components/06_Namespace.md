# Kubernetes Namespace ?

---

In **Kubernetes**, a **Namespace** is a logical partition or virtual cluster within a Kubernetes cluster. It allows you to divide cluster resources among multiple users or projects and helps manage, organize, and isolate resources like pods, services, and deployments.

---

### 🧠 **Why are Namespaces important?**
In large environments (e.g., enterprises), you might have:
- Multiple teams or departments
- Development, testing, and production environments
- A need for resource isolation and access control

Rather than spinning up a separate cluster for each group, **Namespaces** allow sharing a single Kubernetes cluster while maintaining isolation.

---

### 🔧 **How Namespaces Work**
When you create a namespace, you're creating a **scope** for:
- Names of resources (like services and deployments)
- Access control (RBAC)
- Resource quotas and limits

> Note: Resources like nodes, PersistentVolumes, and storage classes **are not namespaced** (they're cluster-wide).

---

### ✅ **Default Namespaces in Kubernetes**
Kubernetes comes with a few predefined namespaces:
| Namespace     | Purpose |
|---------------|---------|
| `default`     | Used when no namespace is specified |
| `kube-system` | System components like kube-dns, kube-proxy |
| `kube-public`| Readable by all users, mostly for cluster info |
| `kube-node-lease` | Heartbeat leases for node health tracking |

---

### 📦 **Creating a Namespace**
```yaml
apiVersion: v1
kind: Namespace
metadata:
  name: dev-environment
```
Apply it:
```bash
kubectl apply -f dev-namespace.yaml
```

---

### 🔍 **Real-World Example: Multi-Tenant SaaS Application**

**Scenario**: You're running a **SaaS application** serving multiple clients (e.g., CloudTechnet, CompanyB). You want to isolate their resources but manage them under a single Kubernetes cluster.

#### 👨‍💼 Each company gets:
- Their own **namespace**
- Their own **deployment**, **services**, and **configmaps**
- Specific **RBAC rules** for developers and admins

#### Example setup:

1. **Namespace for CloudTechnet**
```bash
kubectl create namespace cloudtechnet
```

2. **Deployment for CloudTechnet**
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: web-app
  namespace: cloudtechnet
spec:
  replicas: 3
  selector:
    matchLabels:
      app: web-app
  template:
    metadata:
      labels:
        app: web-app
    spec:
      containers:
      - name: web
        image: cloudtechnet/webapp:latest
        ports:
        - containerPort: 80
```

3. **Resource Quotas**
To prevent overuse of resources:
```yaml
apiVersion: v1
kind: ResourceQuota
metadata:
  name: cloudtechnet-quota
  namespace: cloudtechnet
spec:
  hard:
    pods: "10"
    cpu: "4"
    memory: 8Gi
```

4. **RBAC for CloudTechnet Devs**
Limit access:
```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  namespace: cloudtechnet
  name: dev-role
rules:
- apiGroups: [""]
  resources: ["pods", "services"]
  verbs: ["get", "list", "watch", "create", "delete"]
```

---

### 🛠️ Useful Commands
- List all namespaces:
  ```bash
  kubectl get namespaces
  ```
- Switch context to a namespace:
  ```bash
  kubectl config set-context --current --namespace=cloudtechnet
  ```
- View resources in a specific namespace:
  ```bash
  kubectl get pods -n cloudtechnet
  ```

---

### 🧩 Summary
| Feature | With Namespace |
|---------|----------------|
| Isolation | ✅ |
| Resource control | ✅ |
| Multi-team collaboration | ✅ |
| Single-cluster management | ✅ |

---

