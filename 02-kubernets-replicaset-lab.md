# Kubernetes ReplicaSet Lab Exercises

## Lab 1: Create an NGINX Frontend ReplicaSet

### Objective

Create a ReplicaSet that maintains 3 NGINX web server pods.

### nginx-rs.yaml

```yaml
apiVersion: apps/v1
kind: ReplicaSet
metadata:
  name: nginx-rs
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
        image: nginx
        ports:
        - containerPort: 80
```

### Tasks

```bash
kubectl apply -f nginx-rs.yaml

kubectl get rs
kubectl get po

kubectl describe rs nginx-rs
```

### Validation

```bash
kubectl get po -l app=nginx
```

Expected:

* 3 NGINX pods are running.

---

# Lab 2: Create a PHPMyAdmin ReplicaSet

### Objective

Create a ReplicaSet with 3 PHPMyAdmin pods.

### phpmyadmin-rs.yaml

```yaml
apiVersion: apps/v1
kind: ReplicaSet
metadata:
  name: phpmyadmin-rs
spec:
  replicas: 3
  selector:
    matchLabels:
      app: phpmyadmin
  template:
    metadata:
      labels:
        app: phpmyadmin
    spec:
      containers:
      - name: phpmyadmin
        image: phpmyadmin/phpmyadmin
        ports:
        - containerPort: 80
```

### Tasks

```bash
kubectl apply -f phpmyadmin-rs.yaml

kubectl get rs
kubectl get po

kubectl describe rs phpmyadmin-rs
```

### Validation

```bash
kubectl get po -l app=phpmyadmin
```

Expected:

* 3 PHPMyAdmin pods are running.

---

# Lab 3: Create a Grafana ReplicaSet

### Objective

Create a ReplicaSet with 3 Grafana dashboard pods.

### grafana-rs.yaml

```yaml
apiVersion: apps/v1
kind: ReplicaSet
metadata:
  name: grafana-rs
spec:
  replicas: 3
  selector:
    matchLabels:
      app: grafana
  template:
    metadata:
      labels:
        app: grafana
    spec:
      containers:
      - name: grafana
        image: grafana/grafana
        ports:
        - containerPort: 3000
```

### Tasks

```bash
kubectl apply -f grafana-rs.yaml

kubectl get rs
kubectl get po

kubectl describe rs grafana-rs
```

### Validation

```bash
kubectl get po -l app=grafana
```

Expected:

* 3 Grafana pods are running.

---

# Lab 4: Scale Up ReplicaSets

### Objective

Increase the number of replicas.

### Tasks

```bash
kubectl scale rs nginx-rs --replicas=5

kubectl scale rs phpmyadmin-rs --replicas=4

kubectl scale rs grafana-rs --replicas=6
```

### Validation

```bash
kubectl get rs
kubectl get po
```

Expected:

```text
nginx-rs        5
phpmyadmin-rs   4
grafana-rs      6
```

---

# Lab 5: Test ReplicaSet Self-Healing

### Objective

Delete a pod and verify that ReplicaSet recreates it automatically.

### Tasks

List pods:

```bash
kubectl get po
```

Delete one NGINX pod:

```bash
kubectl delete po <nginx-pod-name>
```

Delete one PHPMyAdmin pod:

```bash
kubectl delete po <phpmyadmin-pod-name>
```

Delete one Grafana pod:

```bash
kubectl delete po <grafana-pod-name>
```

Watch pod recreation:

```bash
kubectl get po -w
```

### Validation

```bash
kubectl get rs
kubectl get po
```

Expected:

* New replacement pods are automatically created.
* Desired replica count remains unchanged.

---

# Lab 6: Scale Down ReplicaSets

### Objective

Reduce the number of replicas.

### Tasks

```bash
kubectl scale rs nginx-rs --replicas=2

kubectl scale rs phpmyadmin-rs --replicas=1

kubectl scale rs grafana-rs --replicas=3
```

### Validation

```bash
kubectl get rs
kubectl get po
```

Expected:

* Replica counts match the new desired state.

---

# Lab 7: Delete ReplicaSets

### Objective

Remove all ReplicaSets and their Pods.

### Tasks

```bash
kubectl delete rs nginx-rs

kubectl delete rs phpmyadmin-rs

kubectl delete rs grafana-rs
```

### Validation

```bash
kubectl get rs

kubectl get po
```

Expected:

* No ReplicaSets exist.
* All associated Pods are removed.

---

## Common Verification Commands

```bash
kubectl get rs

kubectl get po

kubectl get po -o wide

kubectl describe rs <replicaset-name>

kubectl describe po <pod-name>

kubectl logs <pod-name>
```

These labs provide hands-on practice with **ReplicaSet creation, scaling, self-healing, monitoring, and deletion** using real-world applications such as **NGINX, PHPMyAdmin, and Grafana**.
