# Docker Complete Lab Exercise

## Image Pull, Run Containers, Login, Validate, Start/Stop, Remove Containers & Images

---

# Lab Objective

In this lab you will learn:

* Pull Docker images
* Run containers
* Login into containers
* Execute validation commands inside containers
* Exit from containers
* List running/stopped containers
* Start, stop, restart, kill containers
* Remove containers
* Remove images

Containers used:

* Nginx
* HTTPD
* BusyBox
* NodeJS
* Python
* Jenkins
* phpMyAdmin
* MySQL

---

# 1. Login to Linux Server

From Windows PowerShell:

```bash id="xksy6t"
ssh root@<SERVER-IP>
```

Example:

```bash id="lf9j81"
ssh root@192.168.1.100
```

---

# 2. Validate Docker Installation

```bash id="x5vpx1"
docker version
```

```bash id="c6mpbf"
docker info
```

```bash id="2n7t5z"
systemctl status docker
```

---

# 3. Pull Docker Images

## Pull Nginx

```bash id="f1caxg"
docker pull nginx
```

---

## Pull HTTPD

```bash id="h8cz7d"
docker pull httpd
```

---

## Pull BusyBox

```bash id="3wb3f7"
docker pull busybox
```

---

## Pull NodeJS

```bash id="0pwbto"
docker pull node
```

---

## Pull Python

```bash id="8pw70n"
docker pull python
```

---

## Pull Jenkins

```bash id="j4k2va"
docker pull jenkins/jenkins:lts
```

---

## Pull MySQL

```bash id="9q9f91"
docker pull mysql
```

---

## Pull phpMyAdmin

```bash id="7p2kca"
docker pull phpmyadmin/phpmyadmin
```

---

# 4. Verify Downloaded Images

```bash id="n0l1ur"
docker images
```

---

# 5. Run Docker Containers

---

## Run Nginx Container

```bash id="s7n5gc"
docker run -d --name nginx-container -p 8080:80 nginx
```

Verify:

```bash id="9q7vcw"
docker ps
```

Browser Access:

```bash id="a96st8"
http://<SERVER-IP>:8080
```

---

## Run HTTPD Container

```bash id="2k3x6m"
docker run -d --name httpd-container -p 8081:80 httpd
```

Browser Access:

```bash id="4dnbws"
http://<SERVER-IP>:8081
```

---

## Run BusyBox Container

```bash id="4jjwgc"
docker run -dit --name busybox-container busybox
```

---

## Run NodeJS Container

```bash id="3h7s6w"
docker run -dit --name node-container node
```

---

## Run Python Container

```bash id="d1v9tk"
docker run -dit --name python-container python
```

---

## Run Jenkins Container

```bash id="c2tr87"
docker run -d \
--name jenkins-container \
-p 8082:8080 \
-p 50000:50000 \
jenkins/jenkins:lts
```

Browser Access:

```bash id="ysc6rb"
http://<SERVER-IP>:8082
```

---

## Run MySQL Container

```bash id="m0g2tw"
docker run -d \
--name mysql-container \
-e MYSQL_ROOT_PASSWORD=root123 \
-p 3306:3306 \
mysql
```

---

## Run phpMyAdmin Container

```bash id="d8st2s"
docker run -d \
--name phpmyadmin-container \
--link mysql-container:db \
-p 8083:80 \
phpmyadmin/phpmyadmin
```

Browser Access:

```bash id="3aq7ki"
http://<SERVER-IP>:8083
```

---

# 6. List Containers

## Running Containers

```bash id="mp0uk2"
docker ps
```

---

## Running + Stopped Containers

```bash id="5cg50k"
docker ps -a
```

---

# 7. Login into Containers and Validate

---

# Login to Nginx Container

```bash id="v69q5w"
docker exec -it nginx-container bash
```

---

## Run Validation Commands Inside Nginx Container

```bash id="m2n9o7"
hostname
```

```bash id="mjlwm2"
pwd
```

```bash id="e6w9dr"
ls
```

```bash id="zgmj2x"
ps -ef
```

```bash id="0r4n5p"
cat /etc/os-release
```

```bash id="x4sq52"
nginx -v
```

---

## Exit from Nginx Container

```bash id="2g0oj4"
exit
```

---

# Login to HTTPD Container

```bash id="07xw8j"
docker exec -it httpd-container bash
```

---

## Run Validation Commands

```bash id="jlwm2d"
hostname
```

```bash id="j1vb1x"
httpd -v
```

```bash id="7j8fsv"
ps -ef
```

---

## Exit Container

```bash id="u8e4mv"
exit
```

---

# Login to BusyBox Container

```bash id="6wz6ud"
docker exec -it busybox-container sh
```

---

## Run Commands Inside BusyBox

```bash id="f0bg2g"
hostname
```

```bash id="l3rz2n"
pwd
```

```bash id="y9rm6w"
ls
```

```bash id="9bz1s5"
ps
```

```bash id="1y0o6o"
date
```

```bash id="k7k94j"
uname -a
```

```bash id="6rgh0z"
cat /etc/os-release
```

---

## Exit BusyBox Container

```bash id="m5b6ul"
exit
```

---

# Login to NodeJS Container

```bash id="jlwm2e"
docker exec -it node-container bash
```

---

## Validate NodeJS

```bash id="4m2h0t"
node -v
```

```bash id="y0r4bh"
npm -v
```

---

## Exit

```bash id="g0t9h2"
exit
```

---

# Login to Python Container

```bash id="u5m0w3"
docker exec -it python-container bash
```

---

## Validate Python

```bash id="n9d4vx"
python --version
```

```bash id="r6v9zx"
pip --version
```

---

## Exit

```bash id="1gmb5z"
exit
```

---

# Login to Jenkins Container

```bash id="d6kh0x"
docker exec -it jenkins-container bash
```

---

## Validate Jenkins Container

```bash id="9t7d2q"
java -version
```

```bash id="0l1s2r"
ps -ef
```

---

## Exit

```bash id="0s3f1h"
exit
```

---

# Login to MySQL Container

```bash id="g2r6y9"
docker exec -it mysql-container bash
```

---

## Login to MySQL Database

```bash id="a5v1cx"
mysql -u root -p
```

Password:

```bash id="o7h9qp"
root123
```

---

## Validate Database

```sql id="8q6t1n"
show databases;
```

---

## Exit MySQL

```bash id="y8j1vl"
exit
```

---

## Exit Container

```bash id="h6m4rs"
exit
```

---

# 8. View Container Logs

## Nginx Logs

```bash id="q8x9r2"
docker logs nginx-container
```

---

## Jenkins Logs

```bash id="v4z7h0"
docker logs jenkins-container
```

---

# 9. Stop Containers

```bash id="d9k3tp"
docker stop nginx-container
```

```bash id="k0s5m9"
docker stop httpd-container
```

---

# 10. Start Containers

```bash id="9y2n6g"
docker start nginx-container
```

```bash id="6n3v5x"
docker start httpd-container
```

---

# 11. Restart Containers

```bash id="1f8m0q"
docker restart nginx-container
```

---

# 12. Kill Containers

```bash id="8m2p7d"
docker kill busybox-container
```

---

# 13. Remove Containers

---

## Remove Stopped Container

```bash id="3v7s1y"
docker rm busybox-container
```

---

## Remove Multiple Containers

```bash id="2y4z0w"
docker rm nginx-container httpd-container node-container
```

---

## Force Remove Running Container

```bash id="r5g9x1"
docker rm -f python-container
```

---

# 14. Remove Docker Images

---

## Remove Single Image

```bash id="q2m4w7"
docker rmi nginx
```

---

## Remove Multiple Images

```bash id="u9s3e0"
docker rmi httpd busybox node python
```

---

## Force Remove Image

```bash id="n7c8r1"
docker rmi -f mysql
```

---

# 15. Docker Cleanup

---

## Remove Stopped Containers

```bash id="8w6v2m"
docker container prune
```

---

## Remove Unused Images

```bash id="m0z7x4"
docker image prune
```

---

## Remove Everything Unused

```bash id="g9r5k1"
docker system prune -a
```

---

# 16. Student Practice Tasks

---

## Task 1

Pull all images and verify.

```bash id="m7q0z3"
docker images
```

---

## Task 2

Run all containers and verify using:

```bash id="4v8x2n"
docker ps
```

---

## Task 3

Login into:

* nginx-container
* busybox-container
* mysql-container

Run validation commands and exit.

---

## Task 4

Stop and restart containers.

---

## Task 5

Kill one container and verify.

---

## Task 6

Remove:

* one running container
* one stopped container
* one image

---

# 17. Important Docker Commands Summary

| Task               | Command                          |
| ------------------ | -------------------------------- |
| Pull image         | `docker pull nginx`              |
| Run container      | `docker run -d --name web nginx` |
| Running containers | `docker ps`                      |
| All containers     | `docker ps -a`                   |
| Login container    | `docker exec -it web bash`       |
| Stop container     | `docker stop web`                |
| Start container    | `docker start web`               |
| Restart container  | `docker restart web`             |
| Kill container     | `docker kill web`                |
| Remove container   | `docker rm web`                  |
| Remove image       | `docker rmi nginx`               |

---

# 18. Bonus Practice

---

## Run Temporary BusyBox Container

```bash id="hh0v9s"
docker run --rm busybox echo "Hello Docker"
```

---

## Run Interactive Ubuntu Container

```bash id="j8w4x6"
docker run -it ubuntu bash
```

---

## Check Resource Usage

```bash id="m2n1q8"
docker stats
```

---

## Inspect Container Details

```bash id="k5v3r9"
docker inspect nginx-container
```
