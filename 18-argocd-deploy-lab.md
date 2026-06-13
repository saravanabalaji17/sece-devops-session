## Lab Exercise:  Build, Push and Deploy a Python Application using GitLab CI/CD + Docker + Kubernetes + Argo CD (GitOps)

---

# Task 1: Create GitLab Repository

Create a new GitLab project:

```text
Project Name: sece-devops-session
Visibility: Private
```

Clone Repository:

```bash
git clone https://gitlab.com/<username>/sece-devops-session.git

cd sece-devops-session
```

---

# Task 2: Create Application Structure

```bash
mkdir app
mkdir gitlab-k8s-demo
```

Directory Structure:

```text
sece-devops-session/
│
├── app/
│   ├── app.py
│   ├── requirements.txt
│   └── Dockerfile
│
├── gitlab-k8s-demo/
│   ├── python-app-deployment.yaml
│   └── python-app-svc.yaml
│
└── .gitlab-ci.yml
```

---

# Task 3: Create Flask Application

### app/app.py

```python
from flask import Flask

app = Flask(__name__)

@app.route("/")
def home():
    return "Welcome to DevOps Lab"

if __name__ == "__main__":
    app.run(host="0.0.0.0", port=5000)
```

---

### app/requirements.txt

```text
flask
```

---

# Task 4: Create Dockerfile

### app/Dockerfile

```dockerfile
FROM python:3.11-slim

WORKDIR /app

COPY . .

RUN pip install -r requirements.txt

EXPOSE 5000

CMD ["python","app.py"]
```

---

# Task 5: Create Kubernetes Manifests

### gitlab-k8s-demo/python-app-deployment.yaml

```yaml
apiVersion: apps/v1
kind: Deployment

metadata:
  name: my-python-app-deployment

spec:
  replicas: 5

  selector:
    matchLabels:
      app: myapp

  template:
    metadata:
      labels:
        app: myapp

    spec:
      containers:
      - name: my-python-app-container
        image: saravanabalaji/my_python_app:latest

        imagePullPolicy: Always

        ports:
        - containerPort: 5000
```

---

### gitlab-k8s-demo/python-app-svc.yaml

```yaml
apiVersion: v1
kind: Service

metadata:
  name: myapp-service

spec:
  selector:
    app: myapp

  ports:
    - port: 80
      targetPort: 5000

  type: LoadBalancer
```

---

# Task 6: Create GitLab CI/CD Pipeline

Create:

```bash
touch .gitlab-ci.yml
```

Add the following content:

```yaml
stages:
  - build
  - update-manifest

variables:
  IMAGE_NAME: my_python_app

docker_build_job:
  stage: build
  image: docker

  services:
    - docker:dind

  script:
    - docker build -t $IMAGE_NAME:$CI_COMMIT_SHORT_SHA ./app

    - docker images

    - docker tag $IMAGE_NAME:$CI_COMMIT_SHORT_SHA $DOCKER_USERNAME/$IMAGE_NAME:$CI_COMMIT_SHORT_SHA

    - docker login -u $DOCKER_USERNAME -p $DOCKER_PASSWORD

    - docker push $DOCKER_USERNAME/$IMAGE_NAME:$CI_COMMIT_SHORT_SHA

update-manifest:
  stage: update-manifest
  image: alpine:latest

  before_script:
    - apk add --no-cache git sed

    - git config --global user.email "gitlab@example.com"
    - git config --global user.name "GitLab CI"

  script:
    - git checkout main

    - sed -i "s|image: .*|image: $DOCKER_USERNAME/$IMAGE_NAME:$CI_COMMIT_SHORT_SHA|g" gitlab-k8s-demo/python-app-deployment.yaml

    - git add gitlab-k8s-demo/python-app-deployment.yaml

    - git diff --cached --quiet || git commit -m "Update image tag to $CI_COMMIT_SHORT_SHA [skip ci]"

    - git remote set-url origin https://$GITLAB_USERNAME:$GITLAB_TOKEN@gitlab.com/<username>/sece-devops-session.git

    - git push origin main
```

---

# Task 7: Create GitLab Personal Access Token (PAT)

Login to GitLab.

Navigate:

```text
Profile
  → Preferences
  → Access Tokens
```

Create Token:

```text
Token Name: gitlab-cicd-token

Scopes:
✔ api
✔ read_repository
✔ write_repository
```

Click:

```text
Create Personal Access Token
```

Copy and save the token.

Example:

```text
glpat-xxxxxxxxxxxxxxxxxxxxx
```

---

# Task 8: Configure GitLab Variables

Navigate:

```text
Project
  → Settings
  → CI/CD
  → Variables
```

Create Variables:

| Variable        | Example          |
| --------------- | ---------------- |
| DOCKER_USERNAME | saravanabalaji   |
| DOCKER_PASSWORD | DockerHub PAT    |
| GITLAB_USERNAME | saravanabalaji17 |
| GITLAB_TOKEN    | GitLab PAT       |

Validate Variables:

```text
Settings → CI/CD → Variables
```

Expected:

```text
4 Variables Created
```

---

# Task 9: Push Initial Code

```bash
git add .

git commit -m "Initial Commit"

git push origin main
```

---

# Task 10: Validate GitLab Pipeline

Navigate:

```text
GitLab
  → Build
  → Pipelines
```

Expected:

```text
✓ docker_build_job

✓ update-manifest
```

Validate logs:

```text
docker build
docker push
git push
```

All should complete successfully.

---

# Task 11: Install Argo CD

Install Namespace:

```bash
kubectl create namespace argocd
```

Install Argo CD:

```bash
kubectl apply -n argocd \
-f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml
```

Verify:

```bash
kubectl get pods -n argocd
```

Expected:

```text
argocd-server
argocd-repo-server
argocd-application-controller
argocd-dex-server

STATUS = Running
```

---

# Task 12: Expose Argo CD UI

```bash
kubectl patch svc argocd-server \
-n argocd \
-p '{"spec":{"type":"LoadBalancer"}}'
```

Verify:

```bash
kubectl get svc -n argocd
```

Expected:

```text
argocd-server LoadBalancer
```

---

# Task 13: Get Argo CD Admin Password

```bash
kubectl get secret argocd-initial-admin-secret \
-n argocd \
-o jsonpath="{.data.password}" | base64 -d
```

Example:

```text
Xyz123Abc
```

---

# Task 14: Login to Argo CD UI

Open Browser:

```text
https://<ARGOCD-IP>
```

Login:

```text
Username : admin

Password : <decoded password>
```

---

# Task 15: Create Argo CD Application

In Argo CD UI:

```text
Applications
  → New App
```

Configuration:

```text
Application Name:
python-app

Project:
default

Sync Policy:
Automatic

Repository URL:
https://gitlab.com/<username>/sece-devops-session.git

Revision:
HEAD

Path:
gitlab-k8s-demo

Cluster:
https://kubernetes.default.svc

Namespace:
default
```

Click:

```text
Create
```

---

# Task 16: Validate Argo CD Application

Expected UI Status:

```text
Application : python-app

Sync Status : Synced

Health Status : Healthy
```

Validate Resources:

```text
Deployment : Healthy

Service : Healthy
```

---

# Task 17: Validate Kubernetes Deployment

Check Deployment:

```bash
kubectl get deployment
```

Expected:

```text
NAME                       READY
my-python-app-deployment   5/5
```

---

Check Pods:

```bash
kubectl get pods -o wide
```

Expected:

```text
5 Pods Running
```

---

Check Service:

```bash
kubectl get svc
```

Expected:

```text
myapp-service LoadBalancer
```

---

Check Endpoints:

```bash
kubectl get endpoints
```

Expected:

```text
5 Pod IPs Registered
```

---

# Task 18: Test GitOps Flow

Modify:

```python
return "Deployment Triggered Successfully"
```

Commit:

```bash
git add .

git commit -m "Version 2"

git push
```

---

# Validation in GitLab

```text
Build → Pipelines
```

Expected:

```text
Running
 ↓
Passed
```

---

# Validation in Argo CD UI

Observe:

```text
OutOfSync
     ↓
Syncing
     ↓
Synced
```

Then:

```text
Healthy
```

---

# Validation in Kubernetes

Watch rollout:

```bash
kubectl rollout status deployment/my-python-app-deployment
```

Expected:

```text
deployment successfully rolled out
```

Check image:

```bash
kubectl describe deployment my-python-app-deployment | grep Image
```

Expected:

```text
Image: saravanabalaji/my_python_app:<new_commit_id>
```

---

