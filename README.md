# CC Lab 6 – Jenkins and NGINX Load Balancing

## 📌 Overview

This lab demonstrates how modern applications are deployed using:

- Jenkins for automation (CI/CD)
- Docker for containerization
- NGINX for load balancing

We build and deploy multiple backend containers and use NGINX to distribute traffic between them.

---

## 🛠 Technologies Used

- Docker  
- Jenkins  
- NGINX  
- Git & GitHub  

---

# 🚀 Tasks Performed

---

## 🔹 Task 1 – Set Up Jenkins Using Docker

### Objective
Run Jenkins as a Docker container.

### Steps

Pull Jenkins image:
```bash
docker pull jenkins/jenkins:lts
```

Build custom Jenkins image:
```bash
docker build -t jenkins-docker -f Dockerfile.jenkins .
```

Run Jenkins container:
```bash
docker run -d -p 8080:8080 -p 50000:50000 \
-v jenkins_home:/var/jenkins_home \
-v /var/run/docker.sock:/var/run/docker.sock \
--name jenkins jenkins-docker
```

Open in browser:
```
http://localhost:8080
```

Get admin password:
```bash
docker exec jenkins cat /var/jenkins_home/secrets/initialAdminPassword
```

---

## 🔹 Task 2 – Create Jenkins Job to Build Backend

### Objective
Configure Jenkins to build a Docker image from GitHub.

### Git Setup

```bash
git init
git checkout -b main
git remote add origin <your-repository-url>
git add .
git commit -m "Initial Jenkins lab setup"
git push -u origin main
```

### Jenkins Configuration

- Create New Item → Freestyle Project
- Source Code Management → Git
- Branch: `*/main`
- Enable Poll SCM:
```
H/5 * * * *
```

### Build Step (Execute Shell)

```bash
cd CC_LAB-6
docker build -t backend-app backend
```

Click **Build Now**.

---

## 🔹 Task 3 – Parameterized Jenkins Job

### Objective
Deploy 1 or 2 backend containers dynamically.

### Add Choice Parameter

Name:
```
Backend_Count
```

Choices:
```
1
2
```

### Modified Execute Shell

```bash
cd CC_LAB-6
docker rm -f backend1 backend2 || true

if [ "$BACKEND_COUNT" = "1" ]; then
    docker run -d --name backend1 backend-app
else
    docker run -d --name backend1 backend-app
    docker run -d --name backend2 backend-app
fi
```

Run:
- Once with Backend_Count = 1  
- Once with Backend_Count = 2  

---

## 🔹 Task 4 – Jenkins Pipeline

### Objective
Define build and deployment as code.

Create New Item → Pipeline  

Configure:
- Definition: Pipeline script from SCM
- SCM: Git
- Branch: `*/main`

Click **Build Now** and observe Stage View.

---

## 🔹 Task 5 – NGINX Load Balancing

### Round Robin (Default)

```nginx
upstream backend_servers {
    server backend1:8080;
    server backend2:8080;
}

server {
    listen 80;
    location / {
        proxy_pass http://backend_servers;
    }
}
```

---

### Least Connections

```nginx
upstream backend_servers {
    least_conn;
    server backend1:8080;
    server backend2:8080;
}
```

---

### IP Hash

```nginx
upstream backend_servers {
    ip_hash;
    server backend1:8080;
    server backend2:8080;
}
```

---

## 🔎 Testing

Open:
```
http://localhost
```

Refresh multiple times and observe backend changes.

---

## 🛠 Troubleshooting

Check running containers:
```bash
docker ps
```

Check backend logs:
```bash
docker logs backend1
```

If you see **502 Bad Gateway**, verify:
- Containers are running
- Port 8080 is exposed
- All containers are on same Docker network

---

# 🎯 Learning Outcomes

- Understand Jenkins CI/CD automation  
- Deploy Docker containers via Jenkins  
- Use Parameterized builds  
- Implement Jenkins Pipeline  
- Configure NGINX load balancing strategies  

---

# 👨‍💻 Conclusion

This lab demonstrates a simple CI/CD workflow:

GitHub → Jenkins → Docker → NGINX → Browser

It shows how automation and load balancing work together in modern DevOps environments.