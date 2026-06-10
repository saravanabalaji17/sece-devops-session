

# 🧪 Lab: ConfigMap → Deployment → Validation

## 🎯 Goal

Inject configuration into an application using a ConfigMap and verify it inside a running Pod.

---

# 🧱 Step 1: Create ConfigMap

We will create a ConfigMap with simple app settings.

### 📄 configmap.yaml

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: app-config
data:
  APP_COLOR: "blue"
  APP_MODE: "production"
  LOG_LEVEL: "debug"
```

### ▶️ Apply it

```bash
kubectl apply -f configmap.yaml
```

### 🔍 Verify

```bash
kubectl get configmap
kubectl describe configmap app-config
```

---

# 🚀 Step 2: Create Deployment using ConfigMap

We will inject ConfigMap values as **environment variables**.

### 📄 deployment.yaml

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: configmap-demo
spec:
  replicas: 2
  selector:
    matchLabels:
      app: configmap-demo
  template:
    metadata:
      labels:
        app: configmap-demo
    spec:
      containers:
      - name: app-container
        image: busybox
        command: ["sh", "-c", "while true; do env; sleep 3600; done"]
        env:
        - name: APP_COLOR
          valueFrom:
            configMapKeyRef:
              name: app-config
              key: APP_COLOR
        - name: APP_MODE
          valueFrom:
            configMapKeyRef:
              name: app-config
              key: APP_MODE
        - name: LOG_LEVEL
          valueFrom:
            configMapKeyRef:
              name: app-config
              key: LOG_LEVEL
```

### ▶️ Apply Deployment

```bash
kubectl apply -f deployment.yaml
```

---

# 📊 Step 3: Check Pods

```bash
kubectl get pods
```

Pick one pod:

```bash
kubectl get pods -o wide
```

---

# 🔍 Step 4: Validate ConfigMap inside Pod

### Enter the pod

```bash
kubectl exec -it <pod-name> -- sh
```

### Check environment variables

```sh
env | grep APP
```

Expected output:

```
APP_COLOR=blue
APP_MODE=production
```

```sh
env | grep LOG
```

```
LOG_LEVEL=debug
```

---

# 🧪 Step 5: Quick Update Test (Important Concept)

Edit ConfigMap:

```bash
kubectl edit configmap app-config
```

Change:

```
APP_COLOR=green
```

### ⚠️ Important Note

* Env-based ConfigMaps **do NOT auto-refresh in running pods**
* You must restart pods:

```bash
kubectl rollout restart deployment configmap-demo
```

Then validate again.

---

# 📌 Bonus: Mount ConfigMap as File (Optional Lab)

Instead of env variables:

```yaml
volumeMounts:
- name: config-volume
  mountPath: /config
volumes:
- name: config-volume
  configMap:
    name: app-config
```

Then inside pod:

```bash
cat /config/APP_COLOR
```

---

# 🧠 Key Learning

* ConfigMap = external configuration storage
* Can be injected as:

  * Environment variables (static at startup)
  * Files (dynamic file mount)
* Changes require pod restart for env method

---

