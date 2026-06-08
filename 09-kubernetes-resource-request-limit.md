# Kubernetes Lab: Resource Requests and Limits Validation

## Objective

Learn how Kubernetes schedules Pods based on **resource requests** and enforces **resource limits**.

---

# Lab 1: Deployment with Requests Greater Than Node Capacity

## Step 1: Check Node Capacity

```bash
kubectl describe node <node-name>
```

Example output:

```text
Capacity:
  cpu:                2
  memory:             4Gi
```

This node has:

* CPU = 2 cores
* Memory = 4Gi

---

## Step 2: Create Deployment with High Requests

Create `high-request.yaml`

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: high-request-app
spec:
  replicas: 1
  selector:
    matchLabels:
      app: high-request-app
  template:
    metadata:
      labels:
        app: high-request-app
    spec:
      containers:
      - name: nginx
        image: nginx
        resources:
          requests:
            cpu: "4"
            memory: "8Gi"
          limits:
            cpu: "4"
            memory: "8Gi"
```

Apply deployment:

```bash
kubectl apply -f high-request.yaml
```

---

## Step 3: Verify Pod Status

```bash
kubectl get pods
```

Expected:

```text
NAME                                READY   STATUS    RESTARTS   AGE
high-request-app-xxxxx              0/1     Pending   0          1m
```

---

## Step 4: Check Scheduling Events

```bash
kubectl describe pod <pod-name>
```

Expected Event:

```text
Warning  FailedScheduling

0/1 nodes are available:
1 Insufficient cpu.
1 Insufficient memory.
```

---

## Validation

### Check Deployment

```bash
kubectl get deployment
```

### Check ReplicaSet

```bash
kubectl get rs
```

### Check Events

```bash
kubectl get events --sort-by=.metadata.creationTimestamp
```

### Check Resource Requests

```bash
kubectl describe pod <pod-name>
```

Expected:

```text
Requests:
  cpu:      4
  memory:   8Gi
```

---

# Lab 2: Deployment with Valid Requests and Limits

## Step 1: Create Deployment

Create `valid-request.yaml`

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: valid-request-app
spec:
  replicas: 2
  selector:
    matchLabels:
      app: valid-request-app
  template:
    metadata:
      labels:
        app: valid-request-app
    spec:
      containers:
      - name: nginx
        image: nginx
        resources:
          requests:
            cpu: "250m"
            memory: "256Mi"
          limits:
            cpu: "500m"
            memory: "512Mi"
```

Apply deployment:

```bash
kubectl apply -f valid-request.yaml
```

---

## Step 2: Verify Pod Scheduling

```bash
kubectl get pods -o wide
```

Expected:

```text
NAME                                  READY   STATUS    NODE
valid-request-app-xxxxx               1/1     Running
valid-request-app-yyyyy               1/1     Running
```

---

## Step 3: Validate Resource Configuration

```bash
kubectl describe pod <pod-name>
```

Expected:

```text
Requests:
  cpu:      250m
  memory:   256Mi

Limits:
  cpu:      500m
  memory:   512Mi
```

---

## Step 4: Check Deployment Resource Settings

```bash
kubectl get deployment valid-request-app \
-o=jsonpath='{.spec.template.spec.containers[*].resources}'
```

Expected:

```text
map[limits:map[cpu:500m memory:512Mi]
requests:map[cpu:250m memory:256Mi]]
```

---

# Bonus Validation Using Metrics Server

Generate load inside a pod:

```bash
kubectl exec -it deploy/valid-request-app -- sh
```

Install stress utility:

```bash
apk add stress-ng
```

Generate CPU load:

```bash
stress-ng --cpu 2 --timeout 300s
```

Monitor usage:

```bash
kubectl top pod
```

Expected:

```text
NAME                                CPU(cores)   MEMORY(bytes)
valid-request-app-xxxxx             300m         120Mi
```

---

# Expected Learning Outcome

| Scenario                      | Result                           |
| ----------------------------- | -------------------------------- |
| Request > Node Capacity       | Pod remains Pending              |
| Request <= Available Capacity | Pod Scheduled                    |
| Limit Reached                 | CPU throttled / Memory OOMKilled |
| Requests Only                 | Scheduler reserves resources     |
| Limits Only                   | Runtime enforces maximum usage   |

### Useful Commands

```bash
kubectl get pods
kubectl describe pod <pod-name>
kubectl get events
kubectl top node
kubectl top pod
kubectl describe node <node-name>
```

This lab demonstrates the difference between **scheduling decisions (requests)** and **runtime enforcement (limits)** in Kubernetes.
