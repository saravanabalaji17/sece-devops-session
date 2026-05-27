# Docker Lab Exercise – Build Apache Web Image and Push to Registry

## Lab Objective

In this lab you will learn:

* Create a custom Docker image using Dockerfile
* Add custom HTML webpage
* Build Docker image
* Create and validate container
* Access webpage from browser
* Tag image
* Push image to Docker Hub registry
* Pull image from registry and validate

-
---

# Step 1: Create Project Directory

```bash
mkdir apache-docker-lab
cd apache-docker-lab
```

---

# Step 2: Create Dockerfile

Create file:

```bash
vi Dockerfile
```

Paste:

```dockerfile
FROM ubuntu

ENV DEBIAN_FRONTEND=noninteractive

# Install Apache
RUN apt-get update && \
    apt-get install -y apache2 && \
    apt-get clean && \
    rm -rf /var/lib/apt/lists/*

# Copy custom HTML content
COPY index.html /var/www/html/index.html

WORKDIR /var/www/html

# Expose port
EXPOSE 80

# Start Apache
CMD ["/usr/sbin/apache2ctl", "-D", "FOREGROUND"]
```

---

# Step 3: Create Custom HTML Page

Create file:

```bash
vi index.html
```

Paste:

```html
<!DOCTYPE html>
<html>
<head>
    <title>Docker Apache Lab</title>
</head>
<body>
    <h1>Welcome to Docker Apache Container</h1>
    <h2>Apache Web Server Running Successfully</h2>
    <p>Created using Dockerfile</p>
</body>
</html>
```

---

# Step 4: Validate Files

```bash
ls -l
```

Expected:

```bash
Dockerfile
index.html
```

---

# Step 5: Build Docker Image

Syntax:

```bash
docker build -t username/apache-web:v1 .
```

Example:

```bash
docker build -t saravana/apache-web:v1 .
```

Validate:

```bash
docker images
```

Expected:

```bash
REPOSITORY           TAG       IMAGE ID
saravana/apache-web  v1        xxxxxxx
```

---

# Step 6: Create Container

```bash
docker run -d --name apache-container -p 8080:80 saravana/apache-web:v1
```

---

# Step 7: Validate Container

Check running containers:

```bash
docker ps
```

Check logs:

```bash
docker logs apache-container
```

Check inside container:

```bash
docker exec -it apache-container bash
```

Validate Apache process:

```bash
ps -ef | grep apache2
```

Exit:

```bash
exit
```

---

# Step 8: Access Webpage from Browser

Open browser:

```text
http://<SERVER-IP>:8080
```

OR locally:

```text
http://localhost:8080
```

Expected output:

```text
Welcome to Docker Apache Container
```

---

# Step 9: Stop and Start Container

Stop:

```bash
docker stop apache-container
```

Start:

```bash
docker start apache-container
```

Validate:

```bash
docker ps
```

---

# Step 10: Login to Docker Hub

Create account in [Docker Hub](https://hub.docker.com?utm_source=chatgpt.com)

Login:

```bash
docker login
```

Enter:

```text
Docker Hub Username
Docker Hub Password
```

Expected:

```text
Login Succeeded
```

---

# Step 11: Push Image to Registry

Push image:

```bash
docker push saravana/apache-web:v1
```

Validate in Docker Hub repository.

---

# Step 12: Remove Local Image

Stop and delete container:

```bash
docker rm -f apache-container
```

Delete image:

```bash
docker rmi saravana/apache-web:v1
```

Validate:

```bash
docker images
```

---

# Step 13: Pull Image from Registry

```bash
docker pull saravana/apache-web:v1
```

Validate:

```bash
docker images
```

---

# Step 14: Create New Container from Pulled Image

```bash
docker run -d --name apache-container-new -p 9090:80 saravana/apache-web:v1
```

Validate:

```bash
docker ps
```

Access:

```text
http://localhost:9090
```

---

# Step 15: Advanced Validation

Inspect image:

```bash
docker inspect saravana/apache-web:v1
```

Inspect container:

```bash
docker inspect apache-container-new
```

Check mapped ports:

```bash
docker port apache-container-new
```

Check container resource usage:

```bash
docker stats
```

---

# Cleanup

```bash
docker rm -f apache-container-new
docker rmi saravana/apache-web:v1
```

---

# Bonus Tasks

## Task 1

Modify webpage color and rebuild image.

## Task 2

Create image with version:

```bash
docker build -t saravana/apache-web:v2 .
```

## Task 3

Run multiple containers:

```bash
docker run -d --name web1 -p 8081:80 saravana/apache-web:v1
docker run -d --name web2 -p 8082:80 saravana/apache-web:v1
```

## Task 4

Push multiple tags:

```bash
docker tag saravana/apache-web:v1 saravana/apache-web:latest
docker push saravana/apache-web:latest
```

---

