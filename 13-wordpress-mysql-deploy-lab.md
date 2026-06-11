# Lab Exercise: Deploy WordPress with MySQL StatefulSet using local-path StorageClass

## Objective

In this lab, you will:

* Create a namespace
* Create a MySQL StatefulSet
* Use `local-path` StorageClass
* Create Persistent Volume Claims dynamically
* Deploy WordPress
* Expose WordPress using NodePort
* Validate DNS resolution
* Validate persistent storage
* Delete pods and verify data persistence
* Scale StatefulSet and observe behavior


```text
+-------------------+
|    WordPress      |
| Deployment        |
+---------+---------+
          |
          |
          v
+-------------------+
|   mysql Service   |
+---------+---------+
          |
          |
          v
+-------------------+
| MySQL StatefulSet |
| mysql-0           |
+---------+---------+
          |
          |
          v
+-------------------+
| PVC (local-path)  |
+-------------------+
```

---

# Task 1: Verify local-path StorageClass

```bash
kubectl get sc
```

Expected:

```text
NAME                   PROVISIONER
local-path (default)   rancher.io/local-path
```

---

# Task 2: Create Namespace

```yaml
apiVersion: v1
kind: Namespace
metadata:
  name: wordpress
```

Apply:

```bash
kubectl apply -f namespace.yaml
```

Validate:

```bash
kubectl get ns
```

---

# Task 3: Create MySQL Secret

```yaml
apiVersion: v1
kind: Secret
metadata:
  name: mysql-secret
  namespace: wordpress
type: Opaque
stringData:
  MYSQL_ROOT_PASSWORD: root123
```

Apply:

```bash
kubectl apply -f mysql-secret.yaml
```

Validate:

```bash
kubectl get secret -n wordpress
```

---

# Task 4: Create Headless Service for StatefulSet

```yaml
apiVersion: v1
kind: Service
metadata:
  name: mysql
  namespace: wordpress
spec:
  clusterIP: None

  selector:
    app: mysql

  ports:
  - port: 3306
```

Apply:

```bash
kubectl apply -f mysql-headless-svc.yaml
```

Validate:

```bash
kubectl get svc -n wordpress
```

---

# Task 5: Create MySQL StatefulSet

```yaml
apiVersion: apps/v1
kind: StatefulSet
metadata:
  name: mysql
  namespace: wordpress
spec:
  serviceName: mysql

  replicas: 1

  selector:
    matchLabels:
      app: mysql

  template:
    metadata:
      labels:
        app: mysql

    spec:
      containers:
      - name: mysql
        image: mysql:8.0

        ports:
        - containerPort: 3306

        env:
        - name: MYSQL_ROOT_PASSWORD
          valueFrom:
            secretKeyRef:
              name: mysql-secret
              key: MYSQL_ROOT_PASSWORD

        - name: MYSQL_DATABASE
          value: wordpress

        volumeMounts:
        - name: mysql-data
          mountPath: /var/lib/mysql

  volumeClaimTemplates:
  - metadata:
      name: mysql-data

    spec:
      accessModes:
      - ReadWriteOnce

      storageClassName: local-path

      resources:
        requests:
          storage: 5Gi
```

Apply:

```bash
kubectl apply -f mysql-sts.yaml
```

Validate:

```bash
kubectl get sts -n wordpress
kubectl get pvc -n wordpress
kubectl get pods -n wordpress
```

Expected:

```text
mysql-0   Running
```

---

# Task 6: Verify StatefulSet DNS

Launch troubleshooting pod:

```bash
kubectl run ubuntu \
-n wordpress \
-it --rm \
--image=ubuntu:24.04 -- bash
```

Install tools:

```bash
apt update
apt install dnsutils default-mysql-client -y
```

DNS Test:

```bash
nslookup mysql-0.mysql.wordpress.svc.cluster.local
```

Expected:

```text
Name: mysql-0.mysql.wordpress.svc.cluster.local
Address: <pod-ip>
```

---

# Task 7: Connect to MySQL

Inside Ubuntu pod:

```bash
mysql -u root \
-h mysql-0.mysql.wordpress.svc.cluster.local \
-p
```

Password:

```text
root123
```

Validate:

```sql
SHOW DATABASES;
```

Expected:

```text
wordpress
information_schema
mysql
performance_schema
```

---

# Task 8: Create WordPress PVC

```yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: wordpress-pvc
  namespace: wordpress
spec:
  accessModes:
  - ReadWriteOnce

  storageClassName: local-path

  resources:
    requests:
      storage: 5Gi
```

Apply:

```bash
kubectl apply -f wordpress-pvc.yaml
```

---

# Task 9: Deploy WordPress

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: wordpress
  namespace: wordpress

spec:
  replicas: 1

  selector:
    matchLabels:
      app: wordpress

  template:
    metadata:
      labels:
        app: wordpress

    spec:
      containers:
      - name: wordpress
        image: wordpress:latest

        ports:
        - containerPort: 80

        env:
        - name: WORDPRESS_DB_HOST
          value: mysql-0.mysql.wordpress.svc.cluster.local

        - name: WORDPRESS_DB_USER
          value: root

        - name: WORDPRESS_DB_PASSWORD
          value: root123

        - name: WORDPRESS_DB_NAME
          value: wordpress

        volumeMounts:
        - name: wp-data
          mountPath: /var/www/html

      volumes:
      - name: wp-data
        persistentVolumeClaim:
          claimName: wordpress-pvc
```

Apply:

```bash
kubectl apply -f wordpress-deploy.yaml
```

---

# Task 10: Expose WordPress

```yaml
apiVersion: v1
kind: Service
metadata:
  name: wordpress
  namespace: wordpress

spec:
  type: NodePort

  selector:
    app: wordpress

  ports:
  - port: 80
    targetPort: 80
    nodePort: 30080
```

Apply:

```bash
kubectl apply -f wordpress-svc.yaml
```

---

# Task 11: Access WordPress

```bash
kubectl get nodes -o wide
```

Open browser:

```text
http://<NodeIP>:30080
```

Complete WordPress setup.

---

# Task 12: Verify PVCs

```bash
kubectl get pvc -n wordpress
```

Expected:

```text
mysql-data-mysql-0   Bound
wordpress-pvc        Bound
```

---

# Task 13: Verify Persistence

Delete MySQL pod:

```bash
kubectl delete pod mysql-0 -n wordpress
```

Wait:

```bash
kubectl get pods -n wordpress -w
```

Validate:

```bash
mysql -u root \
-h mysql-0.mysql.wordpress.svc.cluster.local \
-p
```

Data should still exist.

---

# Task 14: Verify WordPress Persistence

Delete WordPress pod:

```bash
kubectl delete pod -l app=wordpress -n wordpress
```

New pod should start automatically.

Validate website content remains intact.

---

# Task 15: Scale StatefulSet

Scale to 3:

```bash
kubectl scale sts mysql \
--replicas=3 \
-n wordpress
```

Validate:

```bash
kubectl get pods -n wordpress
```

Expected:

```text
mysql-0
mysql-1
mysql-2
```

Verify DNS:

```bash
nslookup mysql-1.mysql.wordpress.svc.cluster.local
nslookup mysql-2.mysql.wordpress.svc.cluster.local
```

Observe unique PVCs:

```bash
kubectl get pvc -n wordpress
```

Expected:

```text
mysql-data-mysql-0
mysql-data-mysql-1
mysql-data-mysql-2
```

---

# Cleanup

```bash
kubectl delete ns wordpress
```

---

## Learning Outcomes

After completing this lab, you will understand:

* StatefulSet vs Deployment
* Headless Service
* Stable Pod DNS
* Dynamic PVC provisioning
* local-path StorageClass
* Persistent storage validation
* Pod recovery
* Scaling StatefulSets
* WordPress–MySQL integration in Kubernetes
