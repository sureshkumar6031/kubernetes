Creating a **3-Tier Task Manager Application** with **Angular (Frontend)**, **Flask (Python API Backend)**, and **PostgreSQL (Database)** on a **single Ubuntu Server in AWS EC2** is a great full-stack DevOps project. Here's a **complete end-to-end guide** with **code samples, setup instructions**, and **directory structure**.

---

### ✅ OVERALL REQUIREMENT
- **Tier 1**: Angular frontend for UI.
- **Tier 2**: Flask backend REST API to handle business logic.
- **Tier 3**: PostgreSQL to store tasks.
- All components hosted on **a single Ubuntu 22.04 LTS EC2 instance** in AWS.

---

## ✅ STEP 1: Setup EC2 and Install Prerequisites

### 📌 1.1 Launch Ubuntu EC2
- **AMI**: Ubuntu 22.04 LTS
- **Instance type**: t2.micro (or t3.micro)
- **Security Group**:
  - Allow ports: 22 (SSH), 80 (HTTP), 5000 (Flask), 4200 (Angular), 5432 (PostgreSQL)

### 📌 1.2 SSH into EC2
```bash
ssh -i "your-key.pem" ubuntu@your-ec2-public-ip
sudo apt update && sudo apt upgrade -y
```

---

## ✅ STEP 2: Install Required Software

```bash
# Node.js & Angular CLI
curl -fsSL https://deb.nodesource.com/setup_18.x | sudo -E bash -
sudo apt install -y nodejs
sudo npm install -g @angular/cli

# Python & Pip
sudo apt install -y python3 python3-pip python3-venv

# PostgreSQL
sudo apt install -y postgresql postgresql-contrib

# Nginx (optional - for reverse proxy)
sudo apt install -y nginx
```

---

## ✅ STEP 3: Setup PostgreSQL (Tier 3 - Database)

### 📌 3.1 Create Database and User
```bash
sudo -u postgres psql
```

Inside the prompt:
```sql
CREATE DATABASE taskmanager;
CREATE USER taskuser WITH ENCRYPTED PASSWORD 'taskpass';
GRANT ALL PRIVILEGES ON DATABASE taskmanager TO taskuser;
\q
```

---

## ✅ STEP 4: Project Directory Structure

```bash
/home/ubuntu/task-manager-app/
├── backend/           # Flask API
├── frontend/          # Angular App
└── README.md
```

---

## ✅ STEP 5: Flask Backend Setup (Tier 2)

### 📌 5.1 Create Flask Project
```bash
cd ~/task-manager-app
mkdir backend && cd backend
python3 -m venv venv
source venv/bin/activate
pip install flask flask-cors psycopg2-binary
```

### 📌 5.2 `app.py` – Main Flask API

```python
from flask import Flask, request, jsonify
from flask_cors import CORS
import psycopg2

app = Flask(__name__)
CORS(app)

conn = psycopg2.connect(
    dbname="taskmanager",
    user="taskuser",
    password="taskpass",
    host="localhost"
)
cur = conn.cursor()

@app.route("/tasks", methods=["GET"])
def get_tasks():
    cur.execute("SELECT * FROM tasks ORDER BY id;")
    tasks = cur.fetchall()
    return jsonify(tasks)

@app.route("/tasks", methods=["POST"])
def create_task():
    data = request.json
    cur.execute("INSERT INTO tasks (title, status) VALUES (%s, %s) RETURNING id;", 
                (data['title'], 'open'))
    conn.commit()
    return jsonify({"id": cur.fetchone()[0]}), 201

@app.route("/tasks/<int:id>", methods=["PUT"])
def update_task(id):
    data = request.json
    cur.execute("UPDATE tasks SET title=%s, status=%s WHERE id=%s;", 
                (data['title'], data['status'], id))
    conn.commit()
    return '', 204

@app.route("/tasks/<int:id>", methods=["DELETE"])
def delete_task(id):
    cur.execute("DELETE FROM tasks WHERE id=%s;", (id,))
    conn.commit()
    return '', 204

if __name__ == "__main__":
    app.run(host="0.0.0.0", port=5000)
```

### 📌 5.3 PostgreSQL Table Creation
```bash
sudo -u postgres psql -d taskmanager
```
```sql
CREATE TABLE tasks (
    id SERIAL PRIMARY KEY,
    title TEXT NOT NULL,
    status VARCHAR(20) NOT NULL
);
\q
```

### 📌 5.4 Run Flask API
```bash
cd ~/task-manager-app/backend
source venv/bin/activate
python app.py
```

---

## ✅ STEP 6: Angular Frontend Setup (Tier 1)

### 📌 6.1 Create Angular App
```bash
cd ~/task-manager-app
ng new frontend
cd frontend
```

### 📌 6.2 Modify Angular App

#### `src/app/task.service.ts`
```ts
import { Injectable } from '@angular/core';
import { HttpClient } from '@angular/common/http';
import { Observable } from 'rxjs';

@Injectable({ providedIn: 'root' })
export class TaskService {
  private apiUrl = 'http://localhost:5000/tasks';

  constructor(private http: HttpClient) {}

  getTasks(): Observable<any> {
    return this.http.get(this.apiUrl);
  }

  createTask(task: any): Observable<any> {
    return this.http.post(this.apiUrl, task);
  }

  updateTask(id: number, task: any): Observable<any> {
    return this.http.put(`${this.apiUrl}/${id}`, task);
  }

  deleteTask(id: number): Observable<any> {
    return this.http.delete(`${this.apiUrl}/${id}`);
  }
}
```

#### `src/app/app.component.ts`
```ts
import { Component, OnInit } from '@angular/core';
import { TaskService } from './task.service';

@Component({
  selector: 'app-root',
  template: `
    <h1>Task Manager</h1>
    <input [(ngModel)]="newTask" placeholder="New Task" />
    <button (click)="addTask()">Add</button>
    <ul>
      <li *ngFor="let task of tasks">
        {{task[0]}}. {{task[1]}} - {{task[2]}}
        <button (click)="completeTask(task[0])">Complete</button>
        <button (click)="deleteTask(task[0])">Delete</button>
      </li>
    </ul>
  `,
})
export class AppComponent implements OnInit {
  tasks: any[] = [];
  newTask = '';

  constructor(private taskService: TaskService) {}

  ngOnInit() {
    this.loadTasks();
  }

  loadTasks() {
    this.taskService.getTasks().subscribe((data) => (this.tasks = data));
  }

  addTask() {
    this.taskService.createTask({ title: this.newTask }).subscribe(() => {
      this.newTask = '';
      this.loadTasks();
    });
  }

  completeTask(id: number) {
    this.taskService.updateTask(id, { title: 'Completed Task', status: 'completed' })
      .subscribe(() => this.loadTasks());
  }

  deleteTask(id: number) {
    this.taskService.deleteTask(id).subscribe(() => this.loadTasks());
  }
}
```

### 📌 6.3 Install Angular HTTP and Forms
```bash
npm install @angular/common @angular/forms @angular/platform-browser
```

### 📌 6.4 Run Angular App
```bash
ng serve --host 0.0.0.0 --port 4200
```

---

## ✅ STEP 7: Access the App

- **Flask API**: `http://<EC2-PUBLIC-IP>:5000/tasks`
- **Angular UI**: `http://<EC2-PUBLIC-IP>:4200`

---

## ✅ Optional: Production Setup with Nginx

Use Nginx as reverse proxy to:
- Serve Angular from `/`
- Proxy Flask from `/api`

Let me know if you'd like to configure this.

---

## ✅ Summary

### 🎯 Functionality:
- Create, Edit, Complete, Delete tasks with serial number
- Fully connected 3-tier architecture

### ✅ Components:
- PostgreSQL stores data
- Flask REST API handles logic
- Angular displays UI and interacts via HTTP

---

Would you like me to create a GitHub-ready folder with all the code files and upload it? Or would you like to Dockerize this for container-based deployment?