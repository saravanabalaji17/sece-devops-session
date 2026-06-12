### Lab Exercise: Build Python Docker Image and Push to Docker Hub Using GitLab CI/CD

#### Objective

Create a simple Python Flask application, build a Docker image using GitLab CI/CD, and push the image to Docker Hub.

---

## Step 1: Create Project Structure

```text
python-app/
├── app.py
├── requirements.txt
├── Dockerfile
└── .gitlab-ci.yml
```

---

## Step 2: Create Python Application

### app.py

```python
from flask import Flask

app = Flask(__name__)

@app.route('/')
def hello():
    return "Hello from <studentname> Python Docker Application!"

if __name__ == "__main__":
    app.run(host='0.0.0.0', port=5000)
```

---

## Step 3: Create requirements.txt

```text
Flask==2.3.2
```

---

## Step 4: Create Dockerfile

```dockerfile
# Use official Python image
FROM python:3.12-slim

# Set working directory
WORKDIR /app

# Copy requirements file
COPY requirements.txt .

# Install dependencies
RUN pip install --no-cache-dir -r requirements.txt

# Copy application source code
COPY . .

# Expose application port
EXPOSE 5000

# Start application
CMD ["python", "app.py"]
```

---

## Step 5: Create GitLab CI/CD Pipeline

### .gitlab-ci.yml

> Replace **<studentname>** with your Docker Hub username.

```yaml
stages:
  - build

docker_build_job:
  stage: build

  image: docker

  services:
    - docker:dind

  script:
    - docker build -t python-app:v1 .

    - docker images

    - docker tag python-app:v1 <studentname>/python-app:v1

    - docker login -u <studentname> -p <dockerhub-password-or-token>

    - docker push <studentname>/python-app:v1
```

---

## Step 6: Commit and Push Code

```bash
git add .
git commit -m "Initial Commit"
git push origin main
```

---

## Step 7: Verify Pipeline

Navigate to:

```text
GitLab Project
   → Build
      → Pipelines
```

Verify the job completes successfully.

---

## Step 8: Verify Docker Hub Image

Check your Docker Hub repository:

```text
https://hub.docker.com/repositories
```

You should see:

```text
<studentname>/python-app:v1
```

---

## Step 9: Test the Image

Pull the image:

```bash
docker pull <studentname>/python-app:v1
```

Run the container:

```bash
docker run -d -p 5000:5000 --name python-app <studentname>/python-app:v1
```

Verify:

```bash
docker ps
```

Access:

```text
http://<server-ip>:5000
```

Expected Output:

```text
Hello from <studentname> Python Docker Application!
```

---

## Validation Commands

```bash
docker images

docker ps

docker logs python-app

curl http://localhost:5000
```

Expected Output:

```text
Hello from <studentname> Python Docker Application!
```

### Lab Outcome

* Created a Python Flask application.
* Created a Docker image.
* Automated image build using GitLab CI/CD.
* Pushed the image to Docker Hub.
* Pulled and ran the image successfully.
