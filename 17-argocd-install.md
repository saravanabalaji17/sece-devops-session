I assume you mean **Argo CD GUI access (web UI)**. Here is a simple **install + validation lab** focused only on getting UI access working end-to-end.

---

# 🚀 Argo CD Install + GUI Access Validation Lab

## 1. Create namespace

```bash
kubectl create namespace argocd
```

---

## 2. Install Argo CD

```bash
kubectl apply -n argocd -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml
```

Wait until pods are running:

```bash
kubectl get pods -n argocd
```

You should see:

* argocd-server
* argocd-repo-server
* argocd-application-controller

---

## 3. Expose Argo CD UI using LoadBalancer

```bash
kubectl patch svc argocd-server -n argocd \
-p '{"spec": {"type": "LoadBalancer"}}'
```

---

## 4. Get External IP

```bash
kubectl get svc argocd-server -n argocd
```

Wait until:

```
EXTERNAL-IP → <public-ip>
```

---

## 5. Open Argo CD UI (GUI Access)

Open browser:

```
https://<EXTERNAL-IP>
```

⚠️ You may get a certificate warning → click **Advanced → Proceed**

---

## 6. Get Admin Password

```bash
kubectl -n argocd get secret argocd-initial-admin-secret \
-o jsonpath="{.data.password}" | base64 -d
```

---

## 7. Login to Argo CD UI

* Username: `admin`
* Password: (from above command)

---

# ✅ GUI Validation Checklist

After login, confirm:

### ✔ Dashboard opens

You should see Argo CD home screen

### ✔ Cluster is connected

Go to:

```
Settings → Clusters
```

You should see:

* in-cluster (Healthy)

### ✔ Applications page loads

Even if empty, page should open without error

---
