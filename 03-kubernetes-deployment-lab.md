# Kubernetes Deployment Lab Exercise


---

# Lab Environment

Application: **NGINX Web Server**

Initial Image: **nginx:1.25**

Replicas: **3**

---

# Task 1: Create Deployment

### nginx-deployment.yaml

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-deployment
spec:
  replicas: 3
  selector:
    matchLabels:
      app: nginx
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
      - name: nginx
        image: nginx:1.25
        ports:
        - containerPort: 80
```

### Create Deployment

```bash
kubectl apply -f nginx-deployment.yaml
```

### Validation

```bash
kubectl get deploy
kubectl get rs
kubectl get po
kubectl get po -o wide

kubectl describe deploy nginx-deployment
```

Expected:

```text
Deployment: nginx-deployment
Replicas: 3/3
Image: nginx:1.25
```

---

# Task 2: Scale Deployment

### Scale from 3 to 5 Replicas

```bash
kubectl scale deployment nginx-deployment --replicas=5
```

### Validation

```bash
kubectl get deploy
kubectl get rs
kubectl get po
```

Expected:

```text
Replicas: 5/5
```

---

# Task 3: Update Deployment Image

### Check Current Image

```bash
kubectl describe deploy nginx-deployment | grep Image
```

### Update Image

```bash
kubectl set image deployment/nginx-deployment nginx=nginx:1.26
```

### Monitor Rollout

```bash
kubectl rollout status deployment/nginx-deployment
```

### Validation

```bash
kubectl describe deploy nginx-deployment | grep Image

kubectl get rs
kubectl get po
```

Expected:

```text
Image: nginx:1.26
```

Observe:

* New ReplicaSet created.
* Old ReplicaSet scaled down.
* Pods recreated gradually.

---

# Task 4: View Rollout History

### Check Revision History

```bash
kubectl rollout history deployment/nginx-deployment
```

Expected:

```text
REVISION  CHANGE-CAUSE
1
2
```

### View Specific Revision

```bash
kubectl rollout history deployment/nginx-deployment --revision=1

kubectl rollout history deployment/nginx-deployment --revision=2
```

---

# Task 5: Perform Another Update

### Upgrade to Latest Version

```bash
kubectl set image deployment/nginx-deployment nginx=nginx:1.27
```

### Verify Rollout

```bash
kubectl rollout status deployment/nginx-deployment

kubectl rollout history deployment/nginx-deployment
```

Expected:

```text
REVISION
1
2
3
```

---

# Task 6: Rollback Deployment

### Rollback to Previous Revision

```bash
kubectl rollout undo deployment/nginx-deployment
```

### Validation

```bash
kubectl rollout status deployment/nginx-deployment

kubectl describe deploy nginx-deployment | grep Image
```

Expected:

```text
Image: nginx:1.26
```

---

# Task 7: Rollback to Specific Revision

### Check History

```bash
kubectl rollout history deployment/nginx-deployment
```

### Rollback to Revision 1

```bash
kubectl rollout undo deployment/nginx-deployment --to-revision=1
```

### Validation

```bash
kubectl rollout status deployment/nginx-deployment

kubectl describe deploy nginx-deployment | grep Image
```

Expected:

```text
Image: nginx:1.25
```

---

# Task 8: Watch Rolling Update in Real Time

Open Terminal 1:

```bash
kubectl get po -w
```

Open Terminal 2:

```bash
kubectl set image deployment/nginx-deployment nginx=nginx:1.26
```

### Observe

* New Pod created.
* New Pod becomes Ready.
* Old Pod terminated.
* Process repeats until all Pods are updated.

This demonstrates a **Rolling Update** with zero downtime.

---

# Cleanup

```bash
kubectl delete deployment nginx-deployment
```

Verify:

```bash
kubectl get deploy
kubectl get rs
kubectl get po
```

---

# Commands Summary

```bash
# Create Deployment
kubectl apply -f nginx-deployment.yaml

# List Resources
kubectl get deploy
kubectl get rs
kubectl get po

# Scale Deployment
kubectl scale deployment nginx-deployment --replicas=5

# Update Image
kubectl set image deployment/nginx-deployment nginx=nginx:1.26

# Check Rollout
kubectl rollout status deployment/nginx-deployment

# Rollout History
kubectl rollout history deployment/nginx-deployment

# Rollback
kubectl rollout undo deployment/nginx-deployment

# Rollback to Revision
kubectl rollout undo deployment/nginx-deployment --to-revision=1

# Delete Deployment
kubectl delete deployment nginx-deployment
```


