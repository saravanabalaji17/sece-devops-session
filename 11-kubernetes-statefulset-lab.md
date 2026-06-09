# Kubernetes Lab: MySQL StatefulSet with PVC (local-path StorageClass)

This lab covers:

* Create MySQL StatefulSet
* Create PVC using `local-path` StorageClass
* Create Headless Service
* Create Ubuntu client pod
* Install `mysql-client` and `dnsutils`
* Validate StatefulSet DNS
* Delete pod and verify recreation
* Scale up to 3 replicas and scale down to 1
* Connect to MySQL using pod-specific DNS names

---

## 1. Verify StorageClass

```bash
kubectl get sc
```

Expected:

```bash
NAME                   PROVISIONER                    DEFAULT
local-path (default)   rancher.io/local-path          true
```

---

## 2. Create Headless Service

```yaml
# mysql-headless-svc.yaml
apiVersion: v1
kind: Service
metadata:
  name: mysql
spec:
  clusterIP: None
  selector:
    app: mysql
  ports:
  - port: 3306
    name: mysql
```

Apply:

```bash
kubectl apply -f mysql-headless-svc.yaml
```

---

## 3. Create MySQL Secret

```bash
kubectl create secret generic mysql-secret \
--from-literal=password=MySQL@123
```

---

## 4. Create StatefulSet

```yaml
# mysql-sts.yaml
apiVersion: apps/v1
kind: StatefulSet
metadata:
  name: mysql
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

        env:
        - name: MYSQL_ROOT_PASSWORD
          valueFrom:
            secretKeyRef:
              name: mysql-secret
              key: password

        ports:
        - containerPort: 3306

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

---

## 5. Validate StatefulSet

```bash
kubectl get pods
```

Expected:

```bash
mysql-0   Running
```

Check PVC:

```bash
kubectl get pvc
```

Expected:

```bash
mysql-data-mysql-0
```

Check PV:

```bash
kubectl get pv
```

---

## 6. Create Ubuntu Client Pod

```yaml
# ubuntu-client.yaml
apiVersion: v1
kind: Pod
metadata:
  name: ubuntu-client
spec:
  containers:
  - name: ubuntu
    image: ubuntu:24.04
    command:
    - sleep
    - infinity
```

Apply:

```bash
kubectl apply -f ubuntu-client.yaml
```

---

## 7. Install MySQL Client and DNS Tools

Login:

```bash
kubectl exec -it ubuntu-client -- bash
```

Install packages:

```bash
apt update

apt install -y \
default-mysql-client \
dnsutils
```

Verify:

```bash
mysql --version

nslookup kubernetes.default.svc.cluster.local
```

---

# StatefulSet DNS Validation

## Pod DNS

From Ubuntu pod:

```bash
nslookup mysql-0.mysql.default.svc.cluster.local
```

Expected:

```bash
Name: mysql-0.mysql.default.svc.cluster.local
Address: <Pod-IP>
```

---

## Service DNS

```bash
nslookup mysql.default.svc.cluster.local
```

Expected:

```bash
Name: mysql.default.svc.cluster.local
Address: <Service-IP>
```

---

# Connect to MySQL

Retrieve password:

```bash
kubectl get secret mysql-secret \
-o jsonpath='{.data.password}' | base64 -d
```

From Ubuntu pod:

```bash
mysql -u root \
-h mysql-0.mysql.default.svc.cluster.local \
-p
```

Enter:

```text
MySQL@123
```

Verify:

```sql
SHOW DATABASES;
```

---

# Pod Recreation Validation

Delete pod:

```bash
kubectl delete pod mysql-0
```

Watch:

```bash
kubectl get pods -w
```

Expected:

```bash
mysql-0   Terminating
mysql-0   Running
```

Validate hostname remains same:

```bash
kubectl get pod mysql-0
```

Validate DNS:

```bash
nslookup mysql-0.mysql.default.svc.cluster.local
```

---

# Scale StatefulSet to 3

```bash
kubectl scale sts mysql --replicas=3
```

Check:

```bash
kubectl get pods
```

Expected:

```bash
mysql-0
mysql-1
mysql-2
```

PVCs:

```bash
kubectl get pvc
```

Expected:

```bash
mysql-data-mysql-0
mysql-data-mysql-1
mysql-data-mysql-2
```

---

# Validate Individual Pod DNS

From Ubuntu pod:

```bash
nslookup mysql-0.mysql.default.svc.cluster.local

nslookup mysql-1.mysql.default.svc.cluster.local

nslookup mysql-2.mysql.default.svc.cluster.local
```

Each should resolve to a different Pod IP.

---

# Connect to Specific Pods

```bash
mysql -u root \
-h mysql-0.mysql.default.svc.cluster.local \
-p
```

```bash
mysql -u root \
-h mysql-1.mysql.default.svc.cluster.local \
-p
```

```bash
mysql -u root \
-h mysql-2.mysql.default.svc.cluster.local \
-p
```

---

# Scale Down to 1

```bash
kubectl scale sts mysql --replicas=1
```

Verify:

```bash
kubectl get pods
```

Expected:

```bash
mysql-0 Running
```

Check PVCs:

```bash
kubectl get pvc
```

You will still see:

```bash
mysql-data-mysql-0
mysql-data-mysql-1
mysql-data-mysql-2
```

StatefulSets preserve storage even after scale down.

---

# StatefulSet Identity Validation

Check hostname:

```bash
kubectl exec mysql-0 -- hostname
```

Expected:

```bash
mysql-0
```

Check DNS:

```bash
kubectl exec -it ubuntu-client -- \
nslookup mysql-0.mysql.default.svc.cluster.local
```

This demonstrates the core StatefulSet features:

* Stable Pod names (`mysql-0`, `mysql-1`, `mysql-2`)
* Stable DNS records
* Persistent storage per Pod
* Ordered scale up/down
* Data retention after Pod restart and scale down
* Direct Pod access through Headless Service DNS names (`mysql-0.mysql.default.svc.cluster.local`)
