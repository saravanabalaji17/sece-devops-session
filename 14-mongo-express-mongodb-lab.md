# Lab Exercise: Deploy MongoDB and Mongo Express on Kubernetes

## Objective

Deploy:

* MongoDB as a StatefulSet
* Persistent Storage using local-path StorageClass
* Mongo Express as a web-based MongoDB GUI
* Verify database creation and data persistence

---

# Task 1: Create Namespace

```bash
kubectl create namespace mongo
```

Verify:

```bash
kubectl get ns mongo
```

---

# Task 2: Create Secret

Create a secret to store MongoDB credentials.

```yaml
apiVersion: v1
kind: Secret
metadata:
  name: mongo-secret
  namespace: mongo
type: Opaque
stringData:
  mongo-root-username: admin
  mongo-root-password: Password123
```

Apply:

```bash
kubectl apply -f mongo-secret.yaml
```

Verify:

```bash
kubectl get secret -n mongo
kubectl describe secret mongo-secret -n mongo
```

---

# Task 3: Create MongoDB Headless Service

**Important:** Create the Service before the StatefulSet.

```yaml
apiVersion: v1
kind: Service
metadata:
  name: mongo
  namespace: mongo
spec:
  clusterIP: None
  selector:
    app: mongo
  ports:
  - port: 27017
    targetPort: 27017
```

Apply:

```bash
kubectl apply -f mongo-svc.yaml
```

Verify:

```bash
kubectl get svc -n mongo
```

---

# Task 4: Deploy MongoDB StatefulSet

```yaml
apiVersion: apps/v1
kind: StatefulSet
metadata:
  name: mongo
  namespace: mongo
spec:
  serviceName: mongo
  replicas: 1
  selector:
    matchLabels:
      app: mongo

  template:
    metadata:
      labels:
        app: mongo
    spec:
      containers:
      - name: mongo
        image: mongo:7
        ports:
        - containerPort: 27017

        env:
        - name: MONGO_INITDB_ROOT_USERNAME
          valueFrom:
            secretKeyRef:
              name: mongo-secret
              key: mongo-root-username

        - name: MONGO_INITDB_ROOT_PASSWORD
          valueFrom:
            secretKeyRef:
              name: mongo-secret
              key: mongo-root-password

        volumeMounts:
        - name: mongo-data
          mountPath: /data/db

  volumeClaimTemplates:
  - metadata:
      name: mongo-data

    spec:
      accessModes:
      - ReadWriteOnce

      storageClassName: local-path

      resources:
        requests:
          storage: 2Gi
```

Apply:

```bash
kubectl apply -f mongo-sts.yaml
```

Verify:

```bash
kubectl get sts -n mongo
kubectl get pod -n mongo
kubectl get pvc -n mongo
kubectl get pv
```

---

# Task 5: Deploy Mongo Express

### Recommended Hostname

Since MongoDB runs as a StatefulSet:

```yaml
- name: ME_CONFIG_MONGODB_SERVER
  value: mongo-0.mongo
```

or

```yaml
- name: ME_CONFIG_MONGODB_SERVER
  value: mongo
```

For a single replica, `mongo` is usually sufficient.

### Deployment

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: mongo-express
  namespace: mongo
spec:
  replicas: 1

  selector:
    matchLabels:
      app: mongo-express

  template:
    metadata:
      labels:
        app: mongo-express

    spec:
      containers:
      - name: mongo-express
        image: mongo-express:latest

        ports:
        - containerPort: 8081

        env:
        - name: ME_CONFIG_MONGODB_ADMINUSERNAME
          valueFrom:
            secretKeyRef:
              name: mongo-secret
              key: mongo-root-username

        - name: ME_CONFIG_MONGODB_ADMINPASSWORD
          valueFrom:
            secretKeyRef:
              name: mongo-secret
              key: mongo-root-password

        - name: ME_CONFIG_MONGODB_SERVER
          value: mongo

        - name: ME_CONFIG_BASICAUTH_USERNAME
          value: admin

        - name: ME_CONFIG_BASICAUTH_PASSWORD
          value: admin@123
```

Apply:

```bash
kubectl apply -f mongo-express.yaml
```

Verify:

```bash
kubectl get deployment -n mongo
kubectl get pods -n mongo
kubectl logs deployment/mongo-express -n mongo
```

---

# Task 6: Expose Mongo Express

```yaml
apiVersion: v1
kind: Service
metadata:
  name: mongo-express
  namespace: mongo
spec:
  type: NodePort

  selector:
    app: mongo-express

  ports:
  - port: 8081
    targetPort: 8081
    nodePort: 30081
```

Apply:

```bash
kubectl apply -f mongo-express-svc.yaml
```

Verify:

```bash
kubectl get svc -n mongo
```

Access:

```text
http://<NodeIP>:30081
```

Login:

```text
Username: admin
Password: admin@123
```

---

# Task 7: Verify MongoDB Connectivity

Direct connection test:

```bash
kubectl run mongo-client \
--rm -it \
--image=mongo:7 \
--restart=Never \
-n mongo \
-- mongosh mongodb://admin:Password123@mongo:27017/admin
```

---

# Task 8: Create Database

Inside mongosh:

```javascript
show dbs
```

Create database:

```javascript
use companydb
```

Create collection:

```javascript
db.createCollection("employees")
```

Verify:

```javascript
show collections
```

---

# Task 9: Insert Documents

```javascript
db.employees.insertOne({
  name: "Saravana",
  department: "Cloud",
  location: "India"
})
```

Verify:

```javascript
db.employees.find()
```

Pretty output:

```javascript
db.employees.find().pretty()
```

Count documents:

```javascript
db.employees.countDocuments()
```

---

# Task 10: Create Additional Records

```javascript
db.employees.insertMany([
{
  name: "Kumar",
  department: "DevOps",
  location: "India"
},
{
  name: "John",
  department: "Cloud",
  location: "USA"
}
])
```

Verify:

```javascript
db.employees.find()
```

---

# Task 11: Query Documents

Find all:

```javascript
db.employees.find()
```

Find one:

```javascript
db.employees.findOne({name:"Saravana"})
```

Filter:

```javascript
db.employees.find({department:"Cloud"})
```

---

# Task 12: Validate Persistence

Check PVC:

```bash
kubectl get pvc -n mongo
```

Delete MongoDB pod:

```bash
kubectl delete pod mongo-0 -n mongo
```

Wait for recreation:

```bash
kubectl get pod -n mongo -w
```

Reconnect:

```bash
mongosh -u admin -p Password123 --host mongo --authenticationDatabase admin
```

Verify data still exists:

```javascript
use companydb

db.employees.find()
```

Expected result: Data should still be present because it is stored on the Persistent Volume.

---

# Troubleshooting

Check MongoDB logs:

```bash
kubectl logs mongo-0 -n mongo
```

Check Mongo Express logs:

```bash
kubectl logs deployment/mongo-express -n mongo
```

Check DNS resolution:

```bash
kubectl run dns-test \
--rm -it \
--image=busybox:1.36 \
--restart=Never \
-n mongo \
-- nslookup mongo
```

Check service endpoints:

```bash
kubectl get endpoints -n mongo
```

This lab covers StatefulSets, Secrets, Services, Persistent Volumes, MongoDB administration, Mongo Express GUI access, CRUD operations, and persistence validation.
