# Docker Image Creation Lab Exercise

# Objective

* Create custom Docker images using Dockerfile
* Build NGINX image with real HTML website
* Build Python application image
* Build Node.js application image
* Push images to Docker Hub
* Pull images from Docker Hub
* Run containers and validate applications

---

# Lab 1 — Create Custom NGINX Docker Image

## Step 1: Create Project Directory

```bash
mkdir nginx-app
cd nginx-app
```

---

## Step 2: Create HTML Website Data

```bash
vi index.html
```

Add below content:

```html
<!DOCTYPE html>
<html>
<head>
<title>Docker NGINX Website</title>
</head>
<body>
<h1>Welcome to Docker NGINX Container</h1>
<h2>Custom Website using Dockerfile</h2>
<p>This webpage is running inside Docker container.</p>
</body>
</html>
```

Save and exit.

---

## Step 3: Create Dockerfile

```bash
vi Dockerfile
```

Add:

```dockerfile
FROM nginx

COPY index.html /usr/share/nginx/html/index.html
```

Save and exit.

---

## Step 4: Build Docker Image

Replace `yourusername` with your Docker Hub username.

```bash
docker build -t yourusername/custom-nginx:v1 .
```

---

## Step 5: Validate Image

```bash
docker images
```

---

## Step 6: Run Container

```bash
docker run -d \
--name nginx-container \
-p 8080:80 \
yourusername/custom-nginx:v1
```

---

## Step 7: Validate Container

```bash
docker ps
```

---

## Step 8: Access Website

Open browser:

```text
http://<SERVER-IP>:8080
```

Validate custom webpage.

---

# Lab 2 — Create Python Docker Image

## Step 1: Create Project Directory

```bash
mkdir python-app
cd python-app
```

---

## Step 2: Create Python Application

```bash
vi app.py
```

Add:

```python
from flask import Flask

app = Flask(__name__)

@app.route('/')
def hello():
    return "Welcome to Python Docker Application"

if __name__ == '__main__':
    app.run(host='0.0.0.0', port=5000)
```

---

## Step 3: Create Requirements File

```bash
vi requirements.txt
```

Add:

```text
flask
```

---

## Step 4: Create Dockerfile

```bash
vi Dockerfile
```

Add:

```dockerfile
FROM python:3.11

WORKDIR /app

COPY requirements.txt .

RUN pip install -r requirements.txt

COPY app.py .

EXPOSE 5000

CMD ["python", "app.py"]
```

---

## Step 5: Build Docker Image

```bash
docker build -t yourusername/custom-python:v1 .
```

---

## Step 6: Validate Image

```bash
docker images
```

---

## Step 7: Run Container

```bash
docker run -d \
--name python-container \
-p 5000:5000 \
yourusername/custom-python:v1
```

---

## Step 8: Validate Application

Open browser:

```text
http://<SERVER-IP>:5000
```

Validate Python webpage output.

---

# Lab 3 — Create Node.js Docker Image

## Step 1: Create Project Directory

```bash
mkdir node-app
cd node-app
```

---

## Step 2: Create Node.js Application

```bash
vi app.js
```

Add:

```javascript
const http = require('http');

const hostname = '0.0.0.0';
const port = 3000;

const server = http.createServer((req, res) => {
  res.statusCode = 200;
  res.setHeader('Content-Type', 'text/plain');
  res.end('Welcome to Node.js Docker Application');
});

server.listen(port, hostname, () => {
  console.log(`Server running at http://${hostname}:${port}/`);
});
```

---

## Step 3: Create package.json

```bash
vi package.json
```

Add:

```json
{
  "name": "node-app",
  "version": "1.0.0",
  "main": "app.js"
}
```

---

## Step 4: Create Dockerfile

```bash
vi Dockerfile
```

Add:

```dockerfile
FROM node:20

WORKDIR /app

COPY package.json .

COPY app.js .

EXPOSE 3000

CMD ["node", "app.js"]
```

---

## Step 5: Build Docker Image

```bash
docker build -t yourusername/custom-node:v1 .
```

---

## Step 6: Validate Image

```bash
docker images
```

---

## Step 7: Run Container

```bash
docker run -d \
--name node-container \
-p 3000:3000 \
yourusername/custom-node:v1
```

---

## Step 8: Validate Application

Open browser:

```text
http://<SERVER-IP>:3000
```

Validate Node.js webpage output.

---

# Lab 4 — Docker Login

## Step 1: Login to Docker Hub

```bash
docker login
```

Enter:

* Docker Hub username
* Password

---

## Step 2: Validate Login

```bash
cat ~/.docker/config.json
```

---

# Lab 5 — Push Docker Images

## Push NGINX Image

```bash
docker push yourusername/custom-nginx:v1
```

---

## Push Python Image

```bash
docker push yourusername/custom-python:v1
```

---

## Push Node.js Image

```bash
docker push yourusername/custom-node:v1
```

---

# Lab 6 — Validate Images on Docker Hub

## List Local Images

```bash
docker images
```

---

## Open Docker Hub

Validate images are available in repository.

---

# Lab 7 — Pull Images from Docker Hub

## Remove Existing Images

```bash
docker rmi yourusername/custom-nginx:v1
docker rmi yourusername/custom-python:v1
docker rmi yourusername/custom-node:v1
```

---

## Pull Images

```bash
docker pull yourusername/custom-nginx:v1
```

```bash
docker pull yourusername/custom-python:v1
```

```bash
docker pull yourusername/custom-node:v1
```

---

## Validate Pulled Images

```bash
docker images
```

---

# Lab 8 — Run Containers from Pulled Images

## Run NGINX Container

```bash
docker run -d \
--name nginx-container-new \
-p 8081:80 \
yourusername/custom-nginx:v1
```

---

## Run Python Container

```bash
docker run -d \
--name python-container-new \
-p 5001:5000 \
yourusername/custom-python:v1
```

---

## Run Node.js Container

```bash
docker run -d \
--name node-container-new \
-p 3001:3000 \
yourusername/custom-node:v1
```

---

# Validation Commands

## List Images

```bash
docker images
```

---

## List Running Containers

```bash
docker ps
```

---

## View Container Logs

```bash
docker logs nginx-container
```

```bash
docker logs python-container
```

```bash
docker logs node-container
```

---

## Login to Container

```bash
docker exec -it nginx-container bash
```

```bash
docker exec -it python-container bash
```

```bash
docker exec -it node-container bash
```

---

# Cleanup

## Stop Containers

```bash
docker stop nginx-container
docker stop python-container
docker stop node-container
```

---

## Remove Containers

```bash
docker rm nginx-container
docker rm python-container
docker rm node-container
```

---

## Remove Images

```bash
docker rmi yourusername/custom-nginx:v1
docker rmi yourusername/custom-python:v1
docker rmi yourusername/custom-node:v1
```

---

# Interview Questions

1. What is Dockerfile?
2. What is difference between image and container?
3. What is `FROM` instruction?
4. What is `COPY` instruction?
5. Difference between `CMD` and `ENTRYPOINT`?
6. What is `WORKDIR`?
7. What is `EXPOSE`?
8. How to build Docker image?
9. How to push image to Docker Hub?
10. How to pull image from Docker Hub?
11. What is image layer?
12. How Docker caching works?
13. Difference between Alpine and Ubuntu images?
14. How to reduce Docker image size?
15. What is multi-stage build?
