# DevOps Practicals 1–14 — Code & Commands Only

---

## Workflow 1 — Git (Q1 + Q3) · Windows PowerShell + GitHub

### A. Repo + commits
```powershell
mkdir git-practical
cd git-practical
git init -b main
"# Git Practical" | Out-File README.md
git status
git diff
git add README.md
git commit -m "Add project README"
# repeat 2 more edits: git status; git diff; git add; git commit
```

### B. Branches + merges
```powershell
git switch -c feature/login
"Login feature" | Out-File login.txt
git add .
git commit -m "Add login page"
"Login validation" | Add-Content login.txt
git add .
git commit -m "Add login validation"
git switch main
git merge feature/login

git switch -c feature/footer
"Footer" | Out-File footer.txt
git add .
git commit -m "Add footer"
"Footer links" | Add-Content footer.txt
git add .
git commit -m "Add footer links"
git switch main
"Main update" | Out-File main-note.txt
git add .
git commit -m "Update main documentation"
git merge --no-ff feature/footer
```

### C. Faulty commit + revert
```powershell
# make a wrong edit, then:
git add .
git commit -m "Introduce faulty setting"
git log --oneline
git show HEAD
git diff HEAD~1 HEAD
git revert HEAD
git log --oneline --graph --all
```

### D. Push + PR (Q3)
```powershell
git remote add origin https://github.com/USERNAME/REPO.git
git push -u origin main
git switch -c feature/update-readme
# edit README.md 3 times, commit after each
git add README.md
git commit -m "Improve README introduction"
git push -u origin feature/update-readme
```
GitHub: Pull requests → New → base `main`, compare `feature/update-readme` → Create. Add review comment. For conflict: edit same README line on main and feature, push both, fix markers, commit, push.

---

## Workflow 2 — Express Student CRUD API + Postman (Q2 + Q9 + Q11) · Windows

### Setup
```powershell
mkdir student-api
cd student-api
npm init -y
npm install express
```

### app.js
```js
const express = require("express");
const app = express();
app.use(express.json());

let students = [];
let nextId = 1;

app.use((req, res, next) => {
  console.log(req.method, req.path);
  next();
});

app.get("/students", (req, res) => {
  res.json(students);
});

app.get("/students/:id", (req, res) => {
  const student = students.find(s => s.id === Number(req.params.id));
  if (!student) return res.status(404).json({ error: "Student not found" });
  res.json(student);
});

app.post("/students", (req, res) => {
  const { name, roll_number, email } = req.body;
  if (!name || !roll_number || !email) {
    return res.status(400).json({ error: "name, roll_number and email are required" });
  }
  const student = { id: nextId++, name, roll_number, email };
  students.push(student);
  res.status(201).json(student);
});

app.put("/students/:id", (req, res) => {
  const student = students.find(s => s.id === Number(req.params.id));
  if (!student) return res.status(404).json({ error: "Student not found" });
  const { name, roll_number, email } = req.body;
  if (!name || !roll_number || !email) {
    return res.status(400).json({ error: "All fields are required" });
  }
  Object.assign(student, { name, roll_number, email });
  res.json(student);
});

app.delete("/students/:id", (req, res) => {
  const index = students.findIndex(s => s.id === Number(req.params.id));
  if (index === -1) return res.status(404).json({ error: "Student not found" });
  students.splice(index, 1);
  res.json({ message: "Student deleted" });
});

app.listen(5000, () => console.log("API running on port 5000"));
```

### Run
```powershell
node app.js
```

### Postman requests (Body → raw → JSON)
```
POST   http://localhost:5000/students       {"name":"Asha","roll_number":"101","email":"asha@example.com"}  -> 201
GET    http://localhost:5000/students                                                                      -> 200
GET    http://localhost:5000/students/1                                                                    -> 200
PUT    http://localhost:5000/students/1     (all 3 fields)                                                 -> 200
DELETE http://localhost:5000/students/1                                                                    -> 200
POST   http://localhost:5000/students       (missing email)                                                -> 400
GET    http://localhost:5000/students/999                                                                  -> 404
```

### Postman test script (Scripts → Post-response)
```js
pm.test("Status is 201", function () {
  pm.response.to.have.status(201);
});

pm.test("Response contains student id", function () {
  pm.expect(pm.response.json()).to.have.property("id");
});
```
Change 201 → 200 / 400 / 404 per request. Save all in one collection → Collection Runner → Run.

### Push + PR (Q2)
```powershell
git init -b main
git add .
git commit -m "Create Express student API"
git switch -c feature/student-api
# small improvement, then:
git add .
git commit -m "Add student API validation"
git remote add origin https://github.com/USERNAME/REPO.git
git push -u origin feature/student-api
```
Create PR on GitHub: `feature/student-api` → `main`.

---

## Workflow 3 — Docker (Q4 + Q5 + Q6) · Windows + Docker Desktop

### Q4 — docker-web/app.py
```python
from flask import Flask
import os

app = Flask(__name__)

@app.get("/")
def home():
    return {"message": os.getenv("MESSAGE", "Hello from Docker")}

app.run(host="0.0.0.0", port=5000)
```

### requirements.txt
```
flask
```

### Dockerfile (initial)
```dockerfile
FROM python:3.12
WORKDIR /app
COPY . .
RUN pip install flask
EXPOSE 5000
CMD ["python", "app.py"]
```

```powershell
docker build -t webapp:1.0 .
docker run -d --name web1 -p 5000:5000 -e MESSAGE="Hello from practical 4" webapp:1.0
docker ps
curl.exe http://localhost:5000
docker logs web1
```

### Dockerfile (improved)
```dockerfile
FROM python:3.12-slim
WORKDIR /app
COPY requirements.txt .
RUN pip install --no-cache-dir -r requirements.txt
COPY app.py .
EXPOSE 5000
CMD ["python", "app.py"]
```

### .dockerignore
```
.venv
__pycache__
.git
*.pyc
```

```powershell
docker build -t webapp:2.0 .
docker images
docker history webapp:1.0
docker history webapp:2.0
```

### Q5 + Q6 — Lifecycle
```powershell
docker pull nginx:alpine
docker pull redis:7
docker images

docker ps -a
docker run -d --name web1 -p 8081:80 --restart unless-stopped nginx:alpine
docker run -d --name web2 -p 8082:80 nginx:alpine
docker ps
docker ps -a

docker stop web1
docker start web1
docker stop web2
docker rm web2
docker ps -a
```
Browser: http://localhost:8081 and http://localhost:8082

### Failure diagnosis
```powershell
docker run -d --name badweb nginx:alpine wrong-command
docker ps -a
docker logs badweb
docker inspect badweb
docker rm badweb

docker exec -it web1 sh
```

---

## Workflow 4 — Kubernetes (Q7 + Q10) · Killercoda

### deployment.yaml
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: web
spec:
  replicas: 2
  selector:
    matchLabels:
      app: web
  template:
    metadata:
      labels:
        app: web
    spec:
      containers:
      - name: nginx
        image: nginx:alpine
        ports:
        - containerPort: 80
```

### service.yaml
```yaml
apiVersion: v1
kind: Service
metadata:
  name: web-service
spec:
  type: NodePort
  selector:
    app: web
  ports:
  - port: 80
    targetPort: 80
    nodePort: 30080
```

### Apply + verify
```bash
kubectl apply -f deployment.yaml
kubectl apply -f service.yaml
kubectl get deployments
kubectl get pods
kubectl get svc
kubectl describe deployment web
```

### deploy.sh (Q10)
```sh
#!/bin/sh
kubectl apply -f deployment.yaml
kubectl apply -f service.yaml
kubectl get pods
kubectl get svc
```
```bash
chmod +x deploy.sh
./deploy.sh
```

---

## Workflow 5 — Python unittest + GitHub Actions (Q8) · Windows + GitHub

### Structure
```
python-ci/
├── operations.py
├── test_operations.py
└── .github/workflows/python-tests.yml
```

### operations.py
```python
def to_upper(text):
    return text.upper()

def to_lower(text):
    return text.lower()

def reverse_text(text):
    return text[::-1]

def add(a, b):
    return a + b
```

### test_operations.py
```python
import unittest
from operations import to_upper, to_lower, reverse_text, add

class TestOperations(unittest.TestCase):

    def test_upper(self):
        self.assertEqual(to_upper("hello"), "HELLO")

    def test_lower(self):
        self.assertEqual(to_lower("HELLO"), "hello")

    def test_reverse(self):
        self.assertEqual(reverse_text("hello"), "olleh")

    def test_add(self):
        self.assertEqual(add(5, 3), 8)

if __name__ == "__main__":
    unittest.main()
```

### Run locally
```powershell
py -m unittest -v
```

### .github/workflows/python-tests.yml
```yaml
name: Python Tests

on: [push, pull_request]

jobs:
  test:
    runs-on: ubuntu-latest
    steps:
      - name: Checkout code
        uses: actions/checkout@v4

      - name: Set up Python
        uses: actions/setup-python@v5
        with:
          python-version: "3.12"

      - name: Run tests
        run: python -m unittest -v
```

### Push
```powershell
git init
git add .
git commit -m "Add Python functions and unittest tests"
git branch -M main
git remote add origin https://github.com/USERNAME/REPOSITORY.git
git push -u origin main
```
GitHub → Actions → Python Tests (green tick = pass).

---

## Workflow 6 — Grafana API Monitoring (Q12) · Windows + Docker Desktop

### Structure
```
api-monitoring/
├── app.py
├── requirements.txt
├── Dockerfile
├── docker-compose.yml
└── prometheus/prometheus.yml
```

### app.py
```python
from flask import Flask
from prometheus_flask_exporter import PrometheusMetrics

app = Flask(__name__)
metrics = PrometheusMetrics(app)

@app.route("/")
def home():
    return {"message": "API is running"}

@app.route("/hello")
def hello():
    return {"message": "Hello World"}

if __name__ == "__main__":
    app.run(host="0.0.0.0", port=5000)
```

### requirements.txt
```
flask
prometheus-flask-exporter
```

### Dockerfile
```dockerfile
FROM python:3.12-slim
WORKDIR /app
COPY requirements.txt .
RUN pip install -r requirements.txt
COPY app.py .
CMD ["python", "app.py"]
```

### prometheus/prometheus.yml
```yaml
global:
  scrape_interval: 5s
scrape_configs:
  - job_name: flask
    static_configs:
      - targets: ["flask:5000"]
```

### docker-compose.yml
```yaml
services:
  flask:
    build: .
    ports:
      - "5000:5000"
  prometheus:
    image: prom/prometheus
    ports:
      - "9090:9090"
    volumes:
      - ./prometheus/prometheus.yml:/etc/prometheus/prometheus.yml
  grafana:
    image: grafana/grafana
    ports:
      - "3000:3000"
```

### Start
```powershell
docker compose up --build -d
docker compose ps
```
- API: http://localhost:5000/
- Metrics: http://localhost:5000/metrics
- Prometheus targets: http://localhost:9090/targets
- Grafana: http://localhost:3000 (admin/admin)
- Grafana data source URL: `http://prometheus:9090`

### PromQL
```
sum(rate(flask_http_request_total[1m]))
```
```
sum(rate(flask_http_request_duration_seconds_sum[5m]))
/
sum(rate(flask_http_request_duration_seconds_count[5m]))
```
Refresh `/` and `/hello` to generate traffic. If no data, check exact metric names at `/metrics`.

---

## Workflow 7 — Terraform (Q13 + Q14) · Windows local

```powershell
terraform version
mkdir terraform-practical
cd terraform-practical
```

### resource.tf
```hcl
terraform {
  required_providers {
    local = {
      source  = "hashicorp/local"
      version = "2.4"
    }
  }
}

resource "local_file" "hello" {
  filename = "hello.txt"
  content  = "Hello, Terraform"
}

output "file_id" {
  value = local_file.hello.id
}

output "file_name" {
  value = local_file.hello.filename
}
```

### Q13 — Create
```powershell
terraform init
terraform plan
terraform apply
# type: yes
type hello.txt
terraform output
dir
```

### Q14 — Plan + state
```powershell
terraform plan
```
Edit `resource.tf`: `content = "Hello, Terraform Practical 14"`
```powershell
terraform plan
terraform apply
type hello.txt
terraform state list
type terraform.tfstate
```
