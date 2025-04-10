## 🚀 Project-03: Multi-Container Angular + Flask + PostgreSQL Web Application on Kubernetes (On-Prem with `kubeadm`)

### 📘 **Introduction**

This project is a **Task Manager** application built using:

- **Angular** (Frontend)
- **Flask (Python)** (Backend API)
- **PostgreSQL** (Database)

### ✅ **Why This Stack?**

- **Angular** is a widely used frontend framework by enterprises for building dynamic SPAs.
- **Flask** is lightweight, easy to use, and ideal for RESTful API development.
- **PostgreSQL** is a powerful open-source RDBMS used in production environments.

This app helps demonstrate container orchestration of a Python-based web app using Kubernetes, useful for developers moving from monolithic to microservice-based deployments.

---

## 🧰 Prerequisites

- Kubernetes Cluster using `kubeadm` (1 Master, 1 Worker)
- Flannel CNI configured
- `kubectl` access to the cluster
- Docker installed
- DockerHub account or private registry
- Basic Docker, Kubernetes & YAML knowledge

---

## 📁 Project Structure

- Angular Frontend (Port: `4200`)
- Flask Backend API (Port: `8000`)
- PostgreSQL DB (Port: `5432`)

---

## 🧱 Step 1: Docker Images

### 🔹 Angular Frontend - `Dockerfile`

```Dockerfile
# Angular Dockerfile
FROM node:16-alpine as build
WORKDIR /app
COPY package*.json ./
RUN npm install
COPY . .
RUN npm run build --prod

FROM nginx:alpine
COPY --from=build /app/dist/task-app /usr/share/nginx/html
EXPOSE 80
CMD ["nginx", "-g", "daemon off;"]
```

### 🔹 Flask Backend - `Dockerfile`

```Dockerfile
FROM python:3.9-slim
WORKDIR /app
COPY requirements.txt .
RUN pip install -r requirements.txt
COPY . .
EXPOSE 8000
CMD ["python", "app.py"]
```

### 🔹 PostgreSQL

No custom Dockerfile needed, we use:

```bash
docker pull postgres:13
```

---

### 🏗️ Build & Push Images

```bash
docker build -t yourusername/task-frontend:1.0 ./frontend
docker build -t yourusername/task-backend:1.0 ./backend
docker push yourusername/task-frontend:1.0
docker push yourusername/task-backend:1.0
docker push postgres:13
```

---

## 📄 Step 2: Kubernetes YAML Files

### 1️⃣ Namespace

```yaml
apiVersion: v1
kind: Namespace
metadata:
  name: task-app
```

---

### 2️⃣ PostgreSQL Deployment + Service

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: postgres
  namespace: task-app
spec:
  replicas: 1
  selector:
    matchLabels:
      app: postgres
  template:
    metadata:
      labels:
        app: postgres
    spec:
      containers:
        - name: postgres
          image: postgres:13
          ports:
            - containerPort: 5432
          env:
            - name: POSTGRES_USER
              value: taskuser
            - name: POSTGRES_PASSWORD
              value: taskpass
            - name: POSTGRES_DB
              value: taskdb
          volumeMounts:
            - name: postgres-storage
              mountPath: /var/lib/postgresql/data
      volumes:
        - name: postgres-storage
          emptyDir: {}
---
apiVersion: v1
kind: Service
metadata:
  name: postgres
  namespace: task-app
spec:
  ports:
    - port: 5432
  selector:
    app: postgres
```

---

### 3️⃣ Flask Backend Deployment + Service

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: backend
  namespace: task-app
spec:
  replicas: 1
  selector:
    matchLabels:
      app: backend
  template:
    metadata:
      labels:
        app: backend
    spec:
      containers:
        - name: task-backend
          image: yourusername/task-backend:1.0
          ports:
            - containerPort: 8000
          env:
            - name: DB_HOST
              value: postgres
            - name: DB_PORT
              value: "5432"
            - name: DB_USER
              value: taskuser
            - name: DB_PASS
              value: taskpass
            - name: DB_NAME
              value: taskdb
---
apiVersion: v1
kind: Service
metadata:
  name: backend
  namespace: task-app
spec:
  ports:
    - port: 8000
  selector:
    app: backend
```

---

### 4️⃣ Angular Frontend Deployment + NodePort Service

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: frontend
  namespace: task-app
spec:
  replicas: 1
  selector:
    matchLabels:
      app: frontend
  template:
    metadata:
      labels:
        app: frontend
    spec:
      containers:
        - name: task-frontend
          image: yourusername/task-frontend:1.0
          ports:
            - containerPort: 80
          env:
            - name: API_URL
              value: http://backend:8000
---
apiVersion: v1
kind: Service
metadata:
  name: frontend
  namespace: task-app
spec:
  type: NodePort
  ports:
    - port: 80
      nodePort: 30002
  selector:
    app: frontend
```

---

## 🚀 Step 3: Deploy to Kubernetes

```bash
kubectl apply -f namespace.yaml
kubectl apply -f postgres-deployment.yaml
kubectl apply -f backend-deployment.yaml
kubectl apply -f frontend-deployment.yaml
```

---

## 🔍 Step 4: Verify Deployment

```bash
kubectl get pods -n task-app
kubectl get svc -n task-app
```

### Access Frontend:

```http
http://<NodeIP>:30002
```

---

## 🧪 Step 5: Test Functionality

- Add tasks using the Angular frontend
- Data stored in PostgreSQL
- Flask API should show logs in real time
- Verify pod connectivity:
  ```bash
  kubectl exec -it <backend-pod> -n task-app -- curl http://postgres:5432
  ```
- Restart pods if needed:
  ```bash
  kubectl rollout restart deployment/backend -n task-app
  ```

---

## ✅ Expected Output

- Angular frontend accessible on port `30002`
- Flask API connects to PostgreSQL
- Tasks persist in database
- Logs visible from backend and frontend pods

