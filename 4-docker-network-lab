# Docker Network Lab Exercise

## Objective

* Create Docker custom networks
* Run containers inside specific networks
* Assign static IP addresses
* Validate network configuration
* Connect and disconnect containers from networks
* Verify container communication

---

# Lab 1 — Create Docker Network

## Step 1: Create Development Network

```bash
docker network create dev-network
```

## Step 2: List Networks

```bash
docker network ls
```

## Step 3: Inspect Network

```bash
docker network inspect dev-network
```

## Validation

Check:

* Network driver should be `bridge`
* Network should appear in `docker network ls`

---

# Lab 2 — Create Container in Specific Network

## Step 1: Run NGINX Container in dev-network

```bash
docker run -d \
--name network-container \
-p 9091:80 \
--network dev-network \
nginx
```

## Step 2: Verify Running Container

```bash
docker ps
```

## Step 3: Inspect Container

```bash
docker container inspect network-container
```

## Step 4: Inspect Network

```bash
docker network inspect dev-network
```

## Validation

Validate:

* Container is attached to `dev-network`
* Container name appears under Containers section
* Port `9091` is mapped to container port `80`

## Step 5: Access from Browser

```text
http://<SERVER-IP>:9091
```

Validate NGINX webpage opens.

---

# Lab 3 — Create Network with Specific Subnet and Gateway

## Step 1: Create Production Network

```bash
docker network create \
--subnet 192.168.1.0/24 \
--gateway 192.168.1.1 \
prod-network
```

## Step 2: List Networks

```bash
docker network ls
```

## Step 3: Inspect Network

```bash
docker network inspect prod-network
```

## Validation

Check:

* Subnet should be `192.168.1.0/24`
* Gateway should be `192.168.1.1`

---

# Lab 4 — Create Container with Specific IP

## Step 1: Run Container with Static IP

```bash
docker run -d \
--name network-container-1 \
-p 9092:80 \
--network prod-network \
--ip 192.168.1.10 \
nginx
```

## Step 2: Verify Container IP

```bash
docker exec -it network-container-1 hostname -I
```

## Step 3: Inspect Container

```bash
docker container inspect network-container-1
```

## Step 4: Inspect Network

```bash
docker network inspect prod-network
```

## Validation

Validate:

* Container IP should be `192.168.1.10`
* Container should appear inside `prod-network`
* Port `9092` should be mapped correctly

## Step 5: Access from Browser

```text
http://<SERVER-IP>:9092
```

Validate NGINX webpage opens.

---

# Lab 5 — Connect Running Container to Another Network

## Step 1: Connect Container

Connect `network-container` to `prod-network`.

```bash
docker network connect prod-network network-container
```

## Step 2: Verify Network Connection

```bash
docker network inspect prod-network
```

## Step 3: Inspect Container Networks

```bash
docker container inspect network-container
```

## Validation

Validate:

* Container should now belong to both:

  * `dev-network`
  * `prod-network`

---

# Lab 6 — Disconnect Container from Network

## Step 1: Disconnect Container

```bash
docker network disconnect prod-network network-container
```

## Step 2: Verify Disconnection

```bash
docker network inspect prod-network
```

## Validation

Validate:

* `network-container` should no longer appear in `prod-network`

---

# Lab 7 — Validate Container Communication

## Step 1: Create Ubuntu Container in dev-network

```bash
docker run -dit \
--name ubuntu-test \
--network dev-network \
ubuntu
```

## Step 2: Login to Container

```bash
docker exec -it ubuntu-test bash
```

## Step 3: Install Ping Utility

Inside container:

```bash
apt update
apt install iputils-ping -y
```

## Step 4: Ping NGINX Container

```bash
ping network-container
```

OR

```bash
ping 172.18.0.10
```

## Validation

Validate:

* Ping should work successfully
* Containers in same network should communicate

## Step 5: Exit Container

```bash
exit
```

---

# Lab 8 — Cleanup

## Stop Containers

```bash
docker stop network-container
docker stop network-container-1
docker stop ubuntu-test
```

## Remove Containers

```bash
docker rm network-container
docker rm network-container-1
docker rm ubuntu-test
```

## Remove Networks

```bash
docker network rm dev-network
docker network rm prod-network
```

## Validation

```bash
docker ps -a
docker network ls
```

Validate:

* Containers removed
* Custom networks removed

---

# Important Validation Commands

## List Networks

```bash
docker network ls
```

## Inspect Network

```bash
docker network inspect dev-network
```

## Inspect Container

```bash
docker inspect network-container
```

## Show Container IP Address

```bash
docker exec -it network-container hostname -I
```

## Show Network Details Inside Container

```bash
docker exec -it network-container ip addr
```

---

# Interview Questions

1. What is Docker bridge network?
2. What is the default Docker network?
3. Difference between bridge and host network?
4. How do you create custom Docker network?
5. How to assign static IP to container?
6. How containers communicate in same network?
7. What is subnet and gateway in Docker?
8. What is use of `docker network inspect`?
9. How to connect running container to another network?
10. How to disconnect container from network?
11. What is overlay network?
12. What is macvlan network?
13. What happens if IP conflicts occur?
14. Difference between `EXPOSE` and `-p`?
15. How to troubleshoot Docker networking?
