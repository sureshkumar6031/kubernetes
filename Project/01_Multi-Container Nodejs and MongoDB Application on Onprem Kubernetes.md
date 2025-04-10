multi-container Node.js & MongoDB web application on an **On-Prem Kubernetes Cluster** using **kubeadm**. Since you have already set up the master and worker nodes and configured the Flannel CNI, we'll focus on deploying the application.

---

### **1. Prerequisites:**
- Kubernetes Cluster setup using `kubeadm` with one Master and one Worker node.
- Flannel CNI configured.
- Access to both nodes via `kubectl`.
- Docker installed on both nodes.
- Basic knowledge of Docker, Kubernetes, and YAML files.

---

### **2. Project Structure:**
The To-Do application will have:
- React Frontend (port **3000**)
- Node.js Backend (Express API) (port **5000**)
- MongoDB Database (port **27017**)

---

## **Step 1: Prepare Docker Images for Each Component**

### **React Frontend: Dockerfile**
```dockerfile
# React Dockerfile
FROM node:16-alpine
WORKDIR /app
COPY package*.json ./
RUN npm install
COPY . .
RUN npm run build
EXPOSE 3000
CMD ["npm", "start"]
```

### **Node.js Backend: Dockerfile**
```dockerfile
# Node.js Dockerfile
FROM node:16-alpine
WORKDIR /app
COPY package*.json ./
RUN npm install
COPY . .
EXPOSE 5000
CMD ["node", "server.js"]
```

### **MongoDB: Dockerfile**
(MongoDB will use the official image)
```dockerfile
# MongoDB Dockerfile
FROM mongo:latest
```

### **Build Docker Images**
```bash
# On your local machine
docker build -t todo-frontend:1.0 ./frontend
docker build -t todo-backend:1.0 ./backend
docker pull mongo:latest
```

### **Push Docker Images to a Registry**
(Assuming you have a Docker Hub account)
```bash
docker tag todo-frontend:1.0 yourusername/todo-frontend:1.0
docker tag todo-backend:1.0 yourusername/todo-backend:1.0
docker push yourusername/todo-frontend:1.0
docker push yourusername/todo-backend:1.0
docker push mongo:latest
```

---

## **Step 2: Create Kubernetes Configuration Files**

### **1. Namespace:**
```yaml
apiVersion: v1
kind: Namespace
metadata:
  name: todo-app
```

### **2. MongoDB Deployment:**
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: mongo
  namespace: todo-app
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
          image: mongo:latest
          ports:
            - containerPort: 27017
          volumeMounts:
            - name: mongo-persistent-storage
              mountPath: /data/db
      volumes:
        - name: mongo-persistent-storage
          emptyDir: {}
---
apiVersion: v1
kind: Service
metadata:
  name: mongo
  namespace: todo-app
spec:
  ports:
    - port: 27017
      targetPort: 27017
  selector:
    app: mongo
```

---

### **3. Backend Deployment:**
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: backend
  namespace: todo-app
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
        - name: todo-backend
          image: yourusername/todo-backend:1.0
          ports:
            - containerPort: 5000
          env:
            - name: MONGO_URL
              value: "mongodb://mongo:27017/todo"
---
apiVersion: v1
kind: Service
metadata:
  name: backend
  namespace: todo-app
spec:
  ports:
    - port: 5000
      targetPort: 5000
  selector:
    app: backend
```

---

### **4. Frontend Deployment:**
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: frontend
  namespace: todo-app
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
        - name: todo-frontend
          image: yourusername/todo-frontend:1.0
          ports:
            - containerPort: 3000
          env:
            - name: REACT_APP_API_URL
              value: "http://backend:5000"
---
apiVersion: v1
kind: Service
metadata:
  name: frontend
  namespace: todo-app
spec:
  type: NodePort
  ports:
    - port: 3000
      nodePort: 30001
  selector:
    app: frontend
```

---

## **Step 3: Deploy to Kubernetes**

### **Apply Namespace**
```bash
kubectl apply -f namespace.yaml
```

### **Deploy MongoDB**
```bash
kubectl apply -f mongo-deployment.yaml
```

### **Deploy Backend**
```bash
kubectl apply -f backend-deployment.yaml
```

### **Deploy Frontend**
```bash
kubectl apply -f frontend-deployment.yaml
```

---

## **Step 4: Verify Deployment**
```bash
kubectl get pods -n todo-app
kubectl get svc -n todo-app
```

### **Access the Application**
- Access Frontend via NodePort:  
  ```
  http://<NodeIP>:30001
  ```
- Check logs if any issue:
  ```bash
  kubectl logs -n todo-app <pod-name>
  ```

---

## **Step 5: Test the Application**
1. Open the application in a browser.
2. Create, update, and delete tasks.
3. Ensure data persistence in MongoDB.

---

## **Expected Output:**
- The **React Frontend** should be accessible via the **NodePort**.
- The **Node.js API** should correctly interact with the **MongoDB**.
- Adding a task from the frontend should store it in MongoDB and reflect changes immediately.

---

## **Troubleshooting:**
- If the backend fails to connect to MongoDB, check environment variables.
- Check the connectivity between pods using:
  ```bash
  kubectl exec -it <pod-name> -n todo-app -- curl http://mongo:27017
  ```
- Inspect pod logs:
  ```bash
  kubectl logs -n todo-app <pod-name>
  ```
- Restart faulty pods:
  ```bash
  kubectl rollout restart deployment/<deployment-name> -n todo-app
  ```
