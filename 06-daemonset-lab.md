# Kubernetes DaemonSet Lab Exercise: Deploy Node Exporter and Validate

## Lab Objective

Deploy **Prometheus Node Exporter as a DaemonSet**.

You will validate:

* Namespace creation
* DaemonSet deployment
* One node-exporter pod per node
* Host metrics collection
* Port 9100 metrics endpoint
* Troubleshooting DaemonSet

Architecture:

```text
                 Prometheus
                     |
                     |
              node-exporter:9100
                     |
 ------------------------------------------------
 |                    |                         |
Node-1             Node-2                    Node-3

node-exporter      node-exporter             node-exporter

CPU                CPU                       CPU
Memory             Memory                    Memory
Disk               Disk                      Disk
Network            Network                   Network
```

---

# Step 1: Create Namespace

Create monitoring namespace:

```bash
kubectl create namespace monitoring
```

Verify:

```bash
kubectl get namespace
```

Expected:

```
monitoring
```

---

# Step 2: Check Kubernetes Nodes

```bash
kubectl get nodes
```

Example:

```
NAME        STATUS
worker01    Ready
worker02    Ready
worker03    Ready
```

DaemonSet expectation:

```
worker01 --> node-exporter pod
worker02 --> node-exporter pod
worker03 --> node-exporter pod
```

---

# Step 3: Create Node Exporter DaemonSet YAML

Create file:

```bash
cat node-exporter-daemonset.yaml
```

Add:

```yaml
apiVersion: apps/v1
kind: DaemonSet

metadata:

  name: node-exporter

  namespace: monitoring

  labels:

    app: node-exporter


spec:

  selector:

    matchLabels:

      app: node-exporter


  template:


    metadata:

      labels:

        app: node-exporter


    spec:


      hostPID: true

      hostNetwork: true


      containers:


      - name: node-exporter


        image: prom/node-exporter:latest


        ports:


        - containerPort: 9100

          hostPort: 9100

          protocol: TCP



        securityContext:


          privileged: true



        volumeMounts:


        - name: proc

          mountPath: /host/proc

          readOnly: true


        - name: sys

          mountPath: /host/sys

          readOnly: true


        - name: rootfs

          mountPath: /rootfs

          readOnly: true



        args:


        - --path.procfs=/host/proc

        - --path.sysfs=/host/sys

        - --path.rootfs=/rootfs



      volumes:


      - name: proc

        hostPath:

          path: /proc



      - name: sys

        hostPath:

          path: /sys



      - name: rootfs

        hostPath:

          path: /
```

---

# Step 4: Deploy Node Exporter

Apply:

```bash
kubectl apply -f node-exporter-daemonset.yaml
```

Expected:

```
daemonset.apps/node-exporter created
```

---

# Step 5: Check DaemonSet Status

Run:

```bash
kubectl get daemonset node-exporter -n monitoring
```

Expected:

```
NAME            DESIRED   CURRENT   READY
node-exporter   3         3         3
```

Meaning:

```
3 Nodes
 =
3 node-exporter pods
```

---

# Step 6: Check Node Exporter Pods

```bash
kubectl get pods -n monitoring -o wide
```

Example:

```
NAME                 NODE

node-exporter-x1     worker01

node-exporter-x2     worker02

node-exporter-x3     worker03
```

Validate:

```
Every node has one exporter pod
```

---

# Step 7: Check DaemonSet Details

```bash
kubectl describe daemonset node-exporter \
-n monitoring
```

Check:

```
Desired Number of Nodes Scheduled: 3

Current Number of Nodes Scheduled: 3

Number Ready: 3
```

---

# Step 8: Validate Node Exporter Process

Enter pod:

```bash
kubectl exec -it \
node-exporter-xxxxx \
-n monitoring -- sh
```

Check:

```bash
ps -ef
```

Expected:

```
/bin/node_exporter
```

---

# Step 9: Test Metrics Endpoint

Because:

```yaml
hostNetwork: true
hostPort: 9100
```

Node exporter listens on node IP.

From Kubernetes node:

```bash
curl localhost:9100/metrics
```

Expected:

```
# HELP node_cpu_seconds_total
# TYPE node_cpu_seconds_total counter

node_cpu_seconds_total

node_memory_MemTotal_bytes

node_filesystem_size_bytes
```

---

# Step 10: Check CPU Metrics

Inside metrics:

```bash
curl localhost:9100/metrics | grep cpu
```

Example:

```
node_cpu_seconds_total
node_cpu_guest_seconds_total
```

---

# Step 11: Check Memory Metrics

```bash
curl localhost:9100/metrics | grep memory
```

Example:

```
node_memory_MemTotal_bytes
node_memory_MemAvailable_bytes
```

---

# Step 12: Check Disk Metrics

```bash
curl localhost:9100/metrics | grep filesystem
```

Example:

```
node_filesystem_size_bytes
node_filesystem_avail_bytes
```

---

# Step 13: Test Node Failure Scenario

Delete one exporter pod:

```bash
kubectl delete pod \
node-exporter-xxxxx \
-n monitoring
```

Watch:

```bash
kubectl get pods -n monitoring -w
```

Expected:

```
old pod deleted

new pod automatically created
```

Reason:

```
DaemonSet controller maintains desired state
```

---

# Step 14: Add New Worker Node Test

Add worker node:

```
worker04
```

Check:

```bash
kubectl get nodes
```

Then:

```bash
kubectl get pods -n monitoring -o wide
```

Expected:

```
worker04

node-exporter-yyyy
```

No manual deployment needed.

---

# Step 15: Troubleshooting

## Check Events

```bash
kubectl get events \
-n monitoring \
--sort-by=.lastTimestamp
```

---

## Check Pod Logs

```bash
kubectl logs \
node-exporter-xxxxx \
-n monitoring
```

---

## Check Pod Describe

```bash
kubectl describe pod \
node-exporter-xxxxx \
-n monitoring
```

Look for:

```
FailedScheduling

ImagePullBackOff

CrashLoopBackOff
```

---

# Step 16: Cleanup

Remove DaemonSet:

```bash
kubectl delete daemonset node-exporter \
-n monitoring
```

Remove namespace:

```bash
kubectl delete namespace monitoring
```

---

# Final Validation Commands

```bash
kubectl get ds -n monitoring
```

```bash
kubectl get pods -n monitoring -o wide
```

```bash
kubectl describe ds node-exporter -n monitoring
```

```bash
curl localhost:9100/metrics
```

Expected final state:

```
Kubernetes Node
       |
       |
DaemonSet
       |
       |
node-exporter pod
       |
       |
Host CPU / Memory / Disk Metrics
```

