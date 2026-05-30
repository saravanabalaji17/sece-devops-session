# Lab 1: Single Container Nginx Pod

**nginx-pod.yaml**

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: nginx-pod
spec:
  containers:
  - name: nginx
    image: nginx
```

Deploy:

```bash
kubectl create -f nginx-pod.yaml
```

Validate:

```bash
kubectl get po
kubectl get po -o wide
kubectl describe po nginx-pod
kubectl logs nginx-pod
kubectl delete po nginx-pod
kubectl delete -f nginx-pod.yaml
```

---

# Lab 2: Single Container BusyBox Pod

**busybox-pod.yaml**

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: busybox-pod
spec:
  containers:
  - name: busybox
    image: busybox
    command:
    - sh
    - -c
    - while true; do echo "Hello Kubernetes"; sleep 10; done
```

Deploy:

```bash
kubectl create -f busybox-pod.yaml
```

Validate:

```bash
kubectl get po
kubectl logs busybox-pod
kubectl describe po busybox-pod
```

---

# Lab 3: Multi-Container Pod (Nginx + BusyBox)

Both containers share the same Pod IP.

**multi-container-pod.yaml**

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: multi-container-pod
spec:
  containers:
  - name: nginx
    image: nginx

  - name: busybox
    image: busybox
    command:
    - sh
    - -c
    - while true; do sleep 30; done
```

Deploy:

```bash
kubectl create -f multi-container-pod.yaml
```

Validate:

```bash
kubectl get po
kubectl get po -o wide
kubectl describe po multi-container-pod
```

Check logs of specific container:

```bash
kubectl logs multi-container-pod -c nginx

kubectl logs multi-container-pod -c busybox
```

---

# Lab 4: Verify Container-to-Container Communication

Login to BusyBox container:

```bash
kubectl exec -it multi-container-pod -c busybox -- sh
```

Install wget is already available in BusyBox.

Access Nginx running in the same Pod:

```bash
wget -qO- localhost
```

Expected output:

```html
<!DOCTYPE html>
<html>
<head>
<title>Welcome to nginx!</title>
...
```

Why does this work?

```text
Pod
│
├── nginx container (Port 80)
│
└── busybox container
      │
      └── localhost:80
```

Containers inside the same Pod:

* Share the same IP address
* Share the same network namespace
* Can communicate using `localhost`

---

# Lab 5: Multi-Container Logging Pod

**logger-pod.yaml**

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: logger-pod
spec:
  containers:
  - name: app1
    image: busybox
    command:
    - sh
    - -c
    - while true; do echo "Application-1 Running"; sleep 5; done

  - name: app2
    image: busybox
    command:
    - sh
    - -c
    - while true; do echo "Application-2 Running"; sleep 5; done
```

Deploy:

```bash
kubectl create -f logger-pod.yaml
```

View logs:

```bash
kubectl logs logger-pod -c app1

kubectl logs logger-pod -c app2
```

Expected:

```text
Application-1 Running
Application-1 Running
```

```text
Application-2 Running
Application-2 Running
```

---

# Cleanup

```bash
kubectl delete -f nginx-pod.yaml
kubectl delete -f busybox-pod.yaml
kubectl delete -f multi-container-pod.yaml
kubectl delete -f logger-pod.yaml
```

### Key Learning

| Lab   | Concept                                            |
| ----- | -------------------------------------------------- |
| Lab 1 | Single container Pod                               |
| Lab 2 | Pod logs                                           |
| Lab 3 | Multi-container Pod                                |
| Lab 4 | Container-to-container communication via localhost |
| Lab 5 | Container-specific logs using `-c` option          |

These exercises cover the most common Pod interview topics and day-to-day Kubernetes operations without using volumes.
