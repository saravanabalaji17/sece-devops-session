# Kubernetes HostPath Volume Lab Exercise

## Objective

Learn how to:

* Create a Pod using a `hostPath` volume
* Verify data is written to the worker node
* Validate data persistence after Pod deletion
* Recreate Pod and access existing data
* Clean up resources

---

# Understanding HostPath

`hostPath` mounts a directory or file from the Kubernetes node directly into a Pod.

```text
Worker Node
┌─────────────────────────┐
│ /data/hostpath-demo     │
│     hello.txt           │
└──────────┬──────────────┘
           │
           │ hostPath
           ▼
┌─────────────────────────┐
│ Pod                     │
│   /mnt                  │
│   hello.txt             │
└─────────────────────────┘
```

**Important:**

* Data remains on the node even if the Pod is deleted.
* Not recommended for production applications requiring portability.
* Commonly used for labs, testing, and node-level agents.

---

# Step 1: Create a HostPath Pod

Create `hostpath-pod.yaml`

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: hostpath-demo
spec:
  containers:
  - name: app
    image: busybox
    command:
    - sh
    - -c
    - sleep 3600

    volumeMounts:
    - name: host-storage
      mountPath: /mnt

  volumes:
  - name: host-storage
    hostPath:
      path: /data/hostpath-demo
      type: DirectoryOrCreate
```

Apply:

```bash
kubectl apply -f hostpath-pod.yaml
```

Verify:

```bash
kubectl get pod
```

Expected:

```text
hostpath-demo   1/1   Running
```

---

# Step 2: Identify the Node

```bash
kubectl get pod hostpath-demo -o wide
```

Example:

```text
NAME            READY   STATUS
hostpath-demo   1/1     Running

NODE
worker01
```

Note the node name.

---

# Step 3: Verify Mount Inside Pod

```bash
kubectl exec -it hostpath-demo -- sh
```

Inside container:

```bash
mount | grep mnt
```

or

```bash
df -h
```

Exit:

```bash
exit
```

---

# Step 4: Create Data in the Volume

Create a file:

```bash
kubectl exec hostpath-demo -- sh -c \
'echo "Kubernetes HostPath Lab" > /mnt/hello.txt'
```

Verify:

```bash
kubectl exec hostpath-demo -- cat /mnt/hello.txt
```

Expected:

```text
Kubernetes HostPath Lab
```

---

# Step 5: Validate Data on Worker Node

SSH to the worker node hosting the Pod.

Check directory:

```bash
sudo ls -l /data/hostpath-demo
```

Expected:

```text
hello.txt
```

Display contents:

```bash
sudo cat /data/hostpath-demo/hello.txt
```

Expected:

```text
Kubernetes HostPath Lab
```

This confirms data is stored directly on the node.

---

# Step 6: Create Additional Files

```bash
kubectl exec hostpath-demo -- sh
```

Inside container:

```bash
echo "File1" > /mnt/file1.txt
echo "File2" > /mnt/file2.txt
ls -l /mnt
```

Exit:

```bash
exit
```

Verify on node:

```bash
sudo ls -l /data/hostpath-demo
```

Expected:

```text
file1.txt
file2.txt
hello.txt
```

---

# Step 7: Delete the Pod

```bash
kubectl delete pod hostpath-demo
```

Verify:

```bash
kubectl get pods
```

Expected:

```text
No resources found
```

---

# Step 8: Verify Data Still Exists

Check worker node:

```bash
sudo ls -l /data/hostpath-demo
```

Expected:

```text
file1.txt
file2.txt
hello.txt
```

Unlike `emptyDir`, data remains because it is stored on the node filesystem.

---

# Step 9: Recreate the Pod

Apply the same YAML again:

```bash
kubectl apply -f hostpath-pod.yaml
```

Wait for Running state:

```bash
kubectl get pod
```

---

# Step 10: Validate Existing Data

Check files from inside the new Pod:

```bash
kubectl exec hostpath-demo -- ls -l /mnt
```

Expected:

```text
file1.txt
file2.txt
hello.txt
```

Display file:

```bash
kubectl exec hostpath-demo -- cat /mnt/hello.txt
```

Expected:

```text
Kubernetes HostPath Lab
```

This proves the data persisted after Pod deletion.

---

# Step 11: Clean Up

Delete Pod:

```bash
kubectl delete pod hostpath-demo
```

Delete host directory from worker node:

```bash
sudo rm -rf /data/hostpath-demo
```

Verify:

```bash
sudo ls /data
```

Directory should no longer exist.

---

# Validation Checklist

| Validation              | Expected Result          |
| ----------------------- | ------------------------ |
| Pod created             | Running                  |
| Host directory created  | `/data/hostpath-demo`    |
| File created inside Pod | Visible on node          |
| File created on node    | Visible in Pod           |
| Pod deleted             | Data remains             |
| Pod recreated           | Existing data accessible |
| Manual cleanup          | Directory removed        |

---
