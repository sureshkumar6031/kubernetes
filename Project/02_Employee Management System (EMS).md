
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
#**📦 Docker Installation**
# 1. Update package list
sudo apt update

# 2. Install prerequisite packages
sudo apt install apt-transport-https ca-certificates curl software-properties-common -y

# 3. Add Docker’s official GPG key
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /usr/share/keyrings/docker-archive-keyring.gpg

# 4. Add Docker’s official APT repository
echo "deb [arch=$(dpkg --print-architecture) signed-by=/usr/share/keyrings/docker-archive-keyring.gpg] \
https://download.docker.com/linux/ubuntu $(lsb_release -cs) stable" | \
sudo tee /etc/apt/sources.list.d/docker.list > /dev/null

# 5. Update package list again
sudo apt update

# 6. Install Docker Engine
sudo apt install docker-ce docker-ce-cli containerd.io -y

# 7. Start Docker
sudo systemctl start docker

# 8. Enable Docker to start on boot
sudo systemctl enable docker

# 9. (Optional) Run Docker as non-root user
sudo usermod -aG docker $USER
newgrp docker


---
**☸️ Install MicroK8s**
sudo apt install snap
sudo snap install microk8s --classic
sudo microk8s enable dns ingress storage dashboard
sudo microk8s kubectl version
---

**🟢 Node.js & NPM Setup**

# 1. Update and install curl
sudo apt update
sudo apt install curl -y

# 2. Install NVM
curl -o- https://raw.githubusercontent.com/nvm-sh/nvm/v0.39.7/install.sh | bash

# 3. Load NVM into terminal
export NVM_DIR="$HOME/.nvm"
source "$NVM_DIR/nvm.sh"

# 4. Install Node.js version 22
nvm install 22

# 5. Set Node.js 22 as default
nvm use 22
nvm alias default 22

# 6. Verify versions
node -v
npm -v


---
#⚙️ **Angular CLI Setup**
# Install Angular CLI globally
npm install -g @angular/cli

# Check Angular CLI version
ng version

**☕ Spring Boot Setup**
# Install OpenJDK 17
sudo apt update
sudo apt install openjdk-17-jdk -y
java -version

# Clone Spring Boot project
git clone https://github.com/Shraddhasalunke/Employee-Management-System-Angular-Spring-boot.git
cd Employee-Management-System-Angular-Spring-boot

## 🐳 Step 1: Prepare Docker Images

### 🔹 Angular Frontend Dockerfile (frontend/Dockerfile)
```Dockerfile
# Stage 1: Build Angular App
FROM node:18-alpine AS build
WORKDIR /app
COPY package*.json ./
RUN npm install
RUN npm install -g @angular/cli
COPY . .
ENV NODE_OPTIONS=--openssl-legacy-provider
RUN npm run build

# Stage 2: Serve with Nginx
FROM nginx:alpine
COPY --from=build /app/dist/angular-frontend /usr/share/nginx/html
EXPOSE 4200
CMD ["nginx", "-g", "daemon off;"]

```

---
### 🔹 Spring Boot Backend Dockerfile (backend/Dockerfile)
```Dockerfile
# Stage 1: Build the application
FROM maven:3.9.6-eclipse-temurin-17 AS builder
WORKDIR /app
COPY pom.xml .
RUN mvn dependency:go-offline
COPY src ./src
RUN mvn clean package -DskipTests

# Stage 2: Run the application
FROM eclipse-temurin:17-jdk-alpine
WORKDIR /app
COPY --from=builder /app/target/*.jar app.jar
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
docker build -t yourusername/employee-db:1.0 ./db

docker push yourusername/employee-frontend:1.0
docker push yourusername/employee-backend:1.0
docker push yourusername/employee-db:1.0
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
microk8s kubectl apply -f namespace.yaml
microk8s kubectl apply -f mysql-deployment.yaml
microk8s kubectl apply -f backend-deployment.yaml
microk8s kubectl apply -f frontend-deployment.yaml
```

---

## ✅ Step 4: Verify

```bash
microk8s kubectl get pods -n ems-app
microk8s kubectl get svc -n ems-app
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

