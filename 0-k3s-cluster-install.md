
# K3s Cluster Installation Lab

## Architecture

```text
                    +----------------------+
                    |  k3s-master          |
                    |  Control Plane       |
                    |  API Server          |
                    |  Scheduler           |
                    |  Controller Manager  |
                    |  Embedded SQLite DB  |
                    +----------+-----------+
                               |
              -----------------------------------
              |                                 |
              |                                 |
   +----------+-----------+        +-----------+----------+
   | k3s-worker1          |        | k3s-worker2         |
   | kubelet              |        | kubelet             |
   | containerd           |        | containerd          |
   +----------------------+        +---------------------+

          Workloads / Pods Scheduled Here
```

---

## Lab Environment

| Hostname (FQDN)      | Short Name  | IP Address   | Role       |
| -------------------- | ----------- | ------------ | ---------- |
| k3s-master.sece.com  | k3s-master  | 192.168.1.10 | K3s Server |
| k3s-worker1.sece.com | k3s-worker1 | 192.168.1.11 | K3s Agent  |
| k3s-worker2.sece.com | k3s-worker2 | 192.168.1.12 | K3s Agent  |

### VM Sizing

| Node        | vCPU | RAM  | Disk  |
| ----------- | ---- | ---- | ----- |
| k3s-master  | 2    | 4 GB | 30 GB |
| k3s-worker1 | 2    | 2 GB | 20 GB |
| k3s-worker2 | 2    | 2 GB | 20 GB |

---

# Step 1: Configure Hostnames

### Master

```bash
sudo hostnamectl set-hostname k3s-master.sece.com
```

### Worker1

```bash
sudo hostnamectl set-hostname k3s-worker1.sece.com
```

### Worker2

```bash
sudo hostnamectl set-hostname k3s-worker2.sece.com
```

Verify:

```bash
hostname
hostname -f
```

---

# Step 2: Configure /etc/hosts

Run on **all nodes**

```bash
sudo tee -a /etc/hosts <<EOF
192.168.1.10  k3s-master.sece.com  k3s-master
192.168.1.11  k3s-worker1.sece.com k3s-worker1
192.168.1.12  k3s-worker2.sece.com k3s-worker2
EOF
```

Verify:

```bash
ping -c 2 k3s-master.sece.com
ping -c 2 k3s-worker1.sece.com
ping -c 2 k3s-worker2.sece.com
```

---

# Step 3: Update OS

Run on all nodes:

```bash
sudo apt update
sudo apt upgrade -y
```

---

# Step 4: Disable Firewall (Lab Only)

Ubuntu 24.04:

```bash
sudo systemctl stop ufw
sudo systemctl disable ufw
```

Verify:

```bash
sudo ufw status
```

Expected:

```text
Status: inactive
```

---

# Step 5: Install K3s Server

Run on **k3s-master**

```bash
curl -sfL https://get.k3s.io | sh -
```

Verify:

```bash
sudo systemctl status k3s
```

```bash
sudo k3s kubectl get nodes
```

Expected:

```text
NAME                   STATUS   ROLES
k3s-master.sece.com    Ready    control-plane,master
```

---

# Step 6: Get Cluster Join Token

On Master:

```bash
sudo cat /var/lib/rancher/k3s/server/node-token
```

Example:

```text
K10xxxxxxxxxxxxxxxx::server:xxxxxxxxxxxx
```

Copy the token.

---

# Step 7: Install Worker 1

Run on **k3s-worker1**

```bash
curl -sfL https://get.k3s.io | \
K3S_URL=https://k3s-master.sece.com:6443 \
K3S_TOKEN="YOUR_TOKEN" \
sh -
```

Verify:

```bash
sudo systemctl status k3s-agent
```

---

# Step 8: Install Worker 2

Run on **k3s-worker2**

```bash
curl -sfL https://get.k3s.io | \
K3S_URL=https://k3s-master.sece.com:6443 \
K3S_TOKEN="YOUR_TOKEN" \
sh -
```

Verify:

```bash
sudo systemctl status k3s-agent
```

---

# Step 9: Validate Cluster

On Master:

```bash
sudo k3s kubectl get nodes
```

Expected:

```text
NAME                    STATUS   ROLES
k3s-master.sece.com     Ready    control-plane,master
k3s-worker1.sece.com    Ready    <none>
k3s-worker2.sece.com    Ready    <none>
```

Detailed:

```bash
sudo k3s kubectl get nodes -o wide
```

---

# Step 10: Configure kubectl for Normal User

On Master:

```bash
mkdir -p $HOME/.kube

sudo cp /etc/rancher/k3s/k3s.yaml $HOME/.kube/config

sudo chown $USER:$USER $HOME/.kube/config
```

Edit the kubeconfig:

```bash
vi ~/.kube/config
```

Replace:

```yaml
server: https://127.0.0.1:6443
```

With:

```yaml
server: https://k3s-master.sece.com:6443
```

Add to `.bashrc`:

```bash
echo 'export KUBECONFIG=$HOME/.kube/config' >> ~/.bashrc

source ~/.bashrc
```

Verify:

```bash
kubectl get nodes
```

---

# Step 11: Deploy Test Pod

Create file:

### nginx-pod.yaml

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: nginx-pod
  labels:
    app: nginx
spec:
  containers:
  - name: nginx-container
    image: nginx:latest
    ports:
    - containerPort: 80
```

Deploy:

```bash
kubectl apply -f nginx-pod.yaml
```

Verify:

```bash
kubectl get pods

kubectl get pods -o wide

kubectl describe pod nginx-pod

kubectl logs nginx-pod
```

Delete:

```bash
kubectl delete pod nginx-pod
```

---

# Useful Commands

### Cluster

```bash
kubectl get nodes

kubectl get pods -A

kubectl get svc -A

kubectl cluster-info
```

### Node Details

```bash
kubectl describe node k3s-worker1

kubectl describe node k3s-worker2
```

### K3s Service

Master:

```bash
sudo systemctl status k3s

sudo journalctl -u k3s -f
```

Workers:

```bash
sudo systemctl status k3s-agent

sudo journalctl -u k3s-agent -f
```

### Check Pod Placement

```bash
kubectl get pods -o wide
```

