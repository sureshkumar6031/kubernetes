
## 🚀 **Project-02: Employee Management System (EMS)**  

### 🌐 Technologies Used:
- **Frontend**: Angular (Port **4200**)  
- **Backend**: Spring Boot REST API (Port **8080**)  
- **Database**: MySQL (Port **3306**)  

---

## 📌 **Why This Project?**

The **Employee Management System** (EMS) is widely used in organizations to handle:
- Employee records (CRUD)
- Department and role management
- Attendance and performance tracking

This application is chosen because it represents a **real-time enterprise use case**. By containerizing and deploying EMS on Kubernetes, we:
- Showcase **multi-tier application deployment**
- Use **Java-based backend (Spring Boot)** and **Angular frontend**
- Include a **stateful component** (MySQL)  
- Enable DevOps engineers to **practice production-grade deployments**

---

## 🧱 Project Structure

```
employee-management/
├── frontend/         # Angular
│   └── Dockerfile
├── backend/          # Spring Boot
│   └── Dockerfile
└── mysql/            # MySQL uses official image
```

---

## 🐳 Step 1: Prepare Docker Images

### 🔹 Angular Frontend Dockerfile (frontend/Dockerfile)
```Dockerfile
# Angular Dockerfile
FROM node:16-alpine as build
WORKDIR /app
COPY package*.json ./
RUN npm install
COPY . .
RUN npm run build --prod

FROM nginx:alpine
COPY --from=build /app/dist/employee-frontend /usr/share/nginx/html
EXPOSE 4200
CMD ["nginx", "-g", "daemon off;"]
```

---

### 🔹 Spring Boot Backend Dockerfile (backend/Dockerfile)
```Dockerfile
# Spring Boot Dockerfile
FROM openjdk:17-jdk-slim
WORKDIR /app
COPY target/employee-backend.jar app.jar
EXPOSE 8080
ENTRYPOINT ["java", "-jar", "app.jar"]
```

---

### 🔹 MySQL: Use official image
```Dockerfile
FROM mysql:8
```

---

### 🛠️ Build and Push Images

```bash
docker build -t yourusername/employee-frontend:1.0 ./frontend
docker build -t yourusername/employee-backend:1.0 ./backend
docker pull mysql:8

docker push yourusername/employee-frontend:1.0
docker push yourusername/employee-backend:1.0
docker push mysql:8
```

---

## 🧾 Step 2: Kubernetes Configuration Files

### 1️⃣ Namespace
```yaml
apiVersion: v1
kind: Namespace
metadata:
  name: ems-app
```

---

### 2️⃣ MySQL Deployment & Service
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: mysql
  namespace: ems-app
spec:
  replicas: 1
  selector:
    matchLabels:
      app: mysql
  template:
    metadata:
      labels:
        app: mysql
    spec:
      containers:
        - name: mysql
          image: mysql:8
          env:
            - name: MYSQL_ROOT_PASSWORD
              value: rootpass
            - name: MYSQL_DATABASE
              value: emsdb
          ports:
            - containerPort: 3306
          volumeMounts:
            - name: mysql-pv
              mountPath: /var/lib/mysql
      volumes:
        - name: mysql-pv
          emptyDir: {}
---
apiVersion: v1
kind: Service
metadata:
  name: mysql
  namespace: ems-app
spec:
  selector:
    app: mysql
  ports:
    - port: 3306
```

---

### 3️⃣ Spring Boot Backend Deployment & Service
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: backend
  namespace: ems-app
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
        - name: backend
          image: yourusername/employee-backend:1.0
          ports:
            - containerPort: 8080
          env:
            - name: SPRING_DATASOURCE_URL
              value: jdbc:mysql://mysql:3306/emsdb
            - name: SPRING_DATASOURCE_USERNAME
              value: root
            - name: SPRING_DATASOURCE_PASSWORD
              value: rootpass
---
apiVersion: v1
kind: Service
metadata:
  name: backend
  namespace: ems-app
spec:
  selector:
    app: backend
  ports:
    - port: 8080
```

---

### 4️⃣ Angular Frontend Deployment & Service
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: frontend
  namespace: ems-app
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
        - name: frontend
          image: yourusername/employee-frontend:1.0
          ports:
            - containerPort: 80
          env:
            - name: API_URL
              value: "http://backend:8080"
---
apiVersion: v1
kind: Service
metadata:
  name: frontend
  namespace: ems-app
spec:
  type: NodePort
  selector:
    app: frontend
  ports:
    - port: 80
      targetPort: 80
      nodePort: 30002
```

---

## 🚀 Step 3: Deploy to Kubernetes

```bash
kubectl apply -f namespace.yaml
kubectl apply -f mysql-deployment.yaml
kubectl apply -f backend-deployment.yaml
kubectl apply -f frontend-deployment.yaml
```

---

## ✅ Step 4: Verify

```bash
kubectl get pods -n ems-app
kubectl get svc -n ems-app
```

### Access Frontend
```http
http://<NodeIP>:30002
```

---

## 🧪 Step 5: Test EMS App

- Add employees via UI
- Data should persist in MySQL
- Backend APIs should return proper responses
- Frontend should dynamically render data

---

## 🛠️ Troubleshooting

- Check pod logs:
  ```bash
  kubectl logs -n ems-app <pod-name>
  ```

- Check DB connection:
  ```bash
  kubectl exec -it <backend-pod> -n ems-app -- curl mysql:3306
  ```

- Restart failed deployments:
  ```bash
  kubectl rollout restart deployment/backend -n ems-app
  ```

