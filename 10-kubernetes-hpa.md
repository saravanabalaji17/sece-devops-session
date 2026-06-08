# Kubernetes Lab Exercise: NGINX Deployment with HPA (CPU & Memory)

## Objective

* Deploy NGINX
* Configure CPU and Memory requests/limits
* Create Horizontal Pod Autoscaler (HPA)
* Generate CPU and Memory load using `dd`
* Validate scale-up and scale-down

---

# Prerequisites

Verify Metrics Server is installed:

```bash
kubectl top nodes
kubectl top pods
```

Expected:

```bash
NAME          CPU(cores)   MEMORY(bytes)
worker-node   150m         1200Mi
```

---

# Step 1: Create Namespace

```bash
kubectl create namespace hpa-lab
```

---

# Step 2: Create NGINX Deployment

Create `nginx-deployment.yaml`

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx
  namespace: hpa-lab
spec:
  replicas: 1
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
        image: nginx
        resources:
          requests:
            cpu: 100m
            memory: 128Mi
          limits:
            cpu: 500m
            memory: 512Mi
        ports:
        - containerPort: 80
```

Apply:

```bash
kubectl apply -f nginx-deployment.yaml
```

Validate:

```bash
kubectl get pods -n hpa-lab
```

---

# Step 3: Expose Deployment

```bash
kubectl expose deployment nginx \
--port=80 \
--target-port=80 \
--type=ClusterIP \
-n hpa-lab
```

Validate:

```bash
kubectl get svc -n hpa-lab
```

---

# Step 4: Create HPA

Create `nginx-hpa.yaml`

```yaml
apiVersion: autoscaling/v2
kind: HorizontalPodAutoscaler
metadata:
  name: nginx-hpa
  namespace: hpa-lab
spec:
  scaleTargetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: nginx

  minReplicas: 1
  maxReplicas: 10

  metrics:

  - type: Resource
    resource:
      name: cpu
      target:
        type: Utilization
        averageUtilization: 50

  - type: Resource
    resource:
      name: memory
      target:
        type: Utilization
        averageUtilization: 50
```

Apply:

```bash
kubectl apply -f nginx-hpa.yaml
```

Validate:

```bash
kubectl get hpa -n hpa-lab
```

Expected:

```bash
NAME        REFERENCE          TARGETS   MINPODS   MAXPODS
nginx-hpa   Deployment/nginx   0%/50%    1         10
```

---

# Step 5: Generate CPU Load using dd

Find Pod:

```bash
kubectl get pods -n hpa-lab
```

Open shell:

```bash
kubectl exec -it <pod-name> -n hpa-lab -- /bin/bash
```

Run CPU load:

```bash
while true
do
  dd if=/dev/zero of=/dev/null
done
```

Open another terminal.

Check utilization:

```bash
kubectl top pod -n hpa-lab
```

Example:

```bash
NAME                     CPU(cores)
nginx-78d4f4dbf-vjzpk    480m
```

Watch HPA:

```bash
kubectl get hpa -n hpa-lab -w
```

Expected:

```bash
NAME        REFERENCE          TARGETS
nginx-hpa   Deployment/nginx   120%/50%
```

---

# Step 6: Generate Memory Load using dd

Enter pod:

```bash
kubectl exec -it <pod-name> -n hpa-lab -- /bin/bash
```

Allocate memory:

```bash
dd if=/dev/zero of=/tmp/memfile bs=1M count=400
```

Verify:

```bash
ls -lh /tmp/memfile
```

Expected:

```bash
400M
```

Check memory usage:

```bash
kubectl top pod -n hpa-lab
```

Expected:

```bash
NAME                     CPU(cores)   MEMORY(bytes)
nginx-78d4f4dbf-vjzpk    100m         420Mi
```

Watch HPA:

```bash
kubectl get hpa -n hpa-lab -w
```

Expected:

```bash
TARGETS
45%/50%,85%/50%
```

Memory target exceeds 50%, triggering scaling.

---

# Step 7: Validate Scale Up

Watch Pods:

```bash
kubectl get pods -n hpa-lab -w
```

Expected:

```bash
nginx-78d4f4dbf-vjzpk   Running
nginx-78d4f4dbf-abc12   Running
nginx-78d4f4dbf-xyz34   Running
```

Check Deployment:

```bash
kubectl get deploy nginx -n hpa-lab
```

Expected:

```bash
NAME    READY   UP-TO-DATE   AVAILABLE
nginx   3/3     3            3
```

---

# Step 8: Validate HPA

```bash
kubectl describe hpa nginx-hpa -n hpa-lab
```

Look for:

```bash
Metrics:
resource cpu
resource memory

Min replicas: 1
Max replicas: 10

Current replicas: 3
Desired replicas: 3
```

---

# Step 9: Stop Load

Delete memory file:

```bash
rm -f /tmp/memfile
```

Stop CPU load:

```bash
Ctrl + C
```

---

# Step 10: Validate Scale Down

Monitor:

```bash
kubectl get hpa -n hpa-lab -w
```

After a few minutes:

```bash
kubectl get deployment nginx -n hpa-lab
```

Expected:

```bash
NAME    READY
nginx   1/1
```

HPA scales back to the minimum replica count.

---

# Validation Commands Summary

```bash
kubectl top pod -n hpa-lab

kubectl top node

kubectl get hpa -n hpa-lab

kubectl describe hpa nginx-hpa -n hpa-lab

kubectl get deployment nginx -n hpa-lab

kubectl get pods -n hpa-lab

kubectl get events -n hpa-lab --sort-by=.metadata.creationTimestamp
```

### Expected Outcome

* NGINX starts with **1 pod**
* CPU load generated using `dd if=/dev/zero of=/dev/null`
* Memory load generated using `dd if=/dev/zero of=/tmp/memfile`
* HPA scales from **1 → multiple replicas**
* After load removal, HPA scales back to **1 replica** automatically.
