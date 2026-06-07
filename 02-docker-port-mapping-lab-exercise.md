# Docker Port Mapping Lab Exercise

## Objective

* Run containers with Docker port mapping (`-p`)
* Access applications from browser
* Login into containers
* Install packages inside containers
* Deploy frontend website templates
* Validate application access from browser

---

# Lab 1 — Nginx Website Deployment

## Step 1: Pull Nginx Image

```bash
docker pull nginx:latest
```

## Step 2: Run Nginx Container with Port Mapping

```bash
docker run -d --name nginx-container -p 8080:80 nginx:latest
```

## Step 3: Verify Running Container

```bash
docker ps
```

Expected:

* Container should be UP
* Port mapping should show:

```bash
0.0.0.0:8080->80/tcp
```

---

## Step 4: Access Website from Browser

Open browser:

```text
http://<SERVER-IP>:8080
```

OR

```text
http://localhost:8080
```

Expected:

* Default Nginx page should open

---

## Step 5: Login into Container

```bash
docker exec -it nginx-container bash
```

---

## Step 6: Install Required Packages

```bash
apt update
apt install zip unzip wget net-tools -y
```

---

## Step 7: Download Frontend Website Template

```bash
wget -O mywebsite.zip https://templatemo.com/download/templatemo_620_compression
```

OR

```bash
wget -O mywebsite.zip <website_URL>
```

---

## Step 8: Verify Downloaded File

```bash
ls -l
```

---

## Step 9: Unzip Website Files

```bash
unzip mywebsite.zip
```

---

## Step 10: Copy Website Content to Nginx HTML Directory

```bash
cp -rv templatemo_620_compression/* /usr/share/nginx/html/
```

OR

```bash
cp -rv <directory_name>/* /usr/share/nginx/html/
```

---

## Step 11: Exit Container

```bash
exit
```

---

## Step 12: Validate Website

Open browser:

```text
http://<SERVER-IP>:8080
```

Expected:

* Custom frontend website should load

---

# Lab 2 — Apache HTTPD Port Mapping

## Run Apache Container

```bash
docker run -d --name httpd-container -p 9090:80 httpd:latest
```

## Verify

```bash
docker ps
```

## Access from Browser

```text
http://<SERVER-IP>:9090
```

---

# Lab 3 — Tomcat Application Deployment

## Run Tomcat Container

```bash
docker run -d --name tomcat-container -p 7070:8080 tomcat:latest
```

## Verify

```bash
docker ps
```

## Access Tomcat

```text
http://<SERVER-IP>:7070
```

Expected:

* Tomcat default page should open

---

# Lab 4 — PHP Website using PHP Apache Image

## Run PHP Container

```bash
docker run -d --name php-container -p 6060:80 php:8.2-apache
```

## Login into Container

```bash
docker exec -it php-container bash
```

## Create PHP File

```bash
echo "<?php phpinfo(); ?>" > /var/www/html/info.php
```

## Exit

```bash
exit
```

## Access Browser

```text
http://<SERVER-IP>:6060/info.php
```

Expected:

* PHP information page should display

---

# Lab 5 — Multiple Port Mapping Validation

## Run Multiple Containers

```bash
docker run -d --name web1 -p 8001:80 nginx
docker run -d --name web2 -p 8002:80 nginx
docker run -d --name web3 -p 8003:80 nginx
```

## Verify

```bash
docker ps
```

---

## Browser Validation

```text
http://<SERVER-IP>:8001
http://<SERVER-IP>:8002
http://<SERVER-IP>:8003
```

---

# Useful Validation Commands

## Check Port Listening

```bash
netstat -tulnp
```

OR

```bash
ss -tulnp
```

---

## Check Container Logs

```bash
docker logs nginx-container
```

---

## Check Container IP

```bash
docker inspect nginx-container | grep IPAddress
```

---

# Cleanup Exercise

## Stop Containers

```bash
docker stop nginx-container
docker stop httpd-container
docker stop tomcat-container
docker stop php-container
```

---

## Remove Containers

```bash
docker rm nginx-container
docker rm httpd-container
docker rm tomcat-container
docker rm php-container
```

---

# Interview Questions

1. What is Docker port mapping?
2. Difference between EXPOSE and `-p` option?
3. What happens if two containers use same host port?
4. Difference between bridge network and host network?
5. How to check mapped ports in Docker?
6. Difference between container port and host port?
7. What is the use of `docker exec -it`?
8. How to copy files into a running container?
9. How to access containerized applications from browser?
10. How to troubleshoot port access issues in Docker?
