# Docker Volume and Port Mapping Lab Exercise

# Scenario 1 — Nginx Container with Docker Volume Persistence

## Objective

* Create Docker volume
* Mount volume into Nginx container
* Deploy website
* Delete container
* Verify data persistence
* Create new container with same volume
* Validate website from browser

---

# Step 1 — List Existing Volumes

```bash
docker volume ls
```

---

# Step 2 — Create Docker Volume

```bash
docker volume create mydata
```

Expected Output:

```bash
mydata
```

---

# Step 3 — Inspect Docker Volume

```bash
docker volume inspect mydata
```

---

# Step 4 — Verify Docker Volume Path

```bash
ls -l /var/lib/docker/volumes
```

Expected:

* `mydata` directory should exist

---

# Step 5 — Run Nginx Container with Volume and Port Mapping

```bash
docker run -d \
--name nginx-container \
-p 8080:80 \
-v mydata:/usr/share/nginx/html \
nginx:latest
```

Explanation:

* `-p 8080:80` → Port mapping
* `-v mydata:/usr/share/nginx/html` → Persistent storage

---

# Step 6 — Verify Running Container

```bash
docker ps
```

Expected:

```bash
0.0.0.0:8080->80/tcp
```

---

# Step 7 — Access Default Website

Open browser:

```text
http://<SERVER-IP>:8080
```

Expected:

* Default Nginx page should open

---

# Step 8 — Login into Container

```bash
docker exec -it nginx-container bash
```

---

# Step 9 — Install Required Packages

```bash
apt update
apt install zip unzip wget net-tools -y
```

---

# Step 10 — Download Website Template

```bash
wget -O mywebsite.zip https://templatemo.com/download/templatemo_620_compression
```

OR

```bash
wget -O mywebsite.zip <web_site_URL>
```

---

# Step 11 — Verify Downloaded File

```bash
ls -l
```

---

# Step 12 — Extract Website Files

```bash
unzip mywebsite.zip
```

---

# Step 13 — Copy Website Files to Nginx HTML Directory

```bash
cp -rv templatemo_620_compression/* /usr/share/nginx/html/
```

OR

```bash
cp -rv <directory_name>/* /usr/share/nginx/html/
```

---

# Step 14 — Verify Website Files

```bash
ls -l /usr/share/nginx/html/
```

---

# Step 15 — Exit Container

```bash
exit
```

---

# Step 16 — Validate Website from Browser

Open browser:

```text
http://<SERVER-IP>:8080
```

Expected:

* Custom frontend website should load

---

# Step 17 — Verify Data on Docker Host

```bash
cd /var/lib/docker/volumes/mydata/_data
ls -l
```

Expected:

* Website files should exist

---

# Step 18 — Stop Container

```bash
docker stop nginx-container
```

---

# Step 19 — Remove Container

```bash
docker rm nginx-container
```

---

# Step 20 — Verify Volume Still Exists

```bash
docker volume ls
```

Expected:

* `mydata` volume should still exist

---

# Step 21 — Verify Data Still Exists on Host

```bash
cd /var/lib/docker/volumes/mydata/_data
ls -l
```

Expected:

* Website files should still exist

---

# Step 22 — Create New Container with Same Volume and New Port

```bash
docker run -d \
--name nginx-new-container \
-p 9090:80 \
-v mydata:/usr/share/nginx/html \
nginx:latest
```

---

# Step 23 — Verify Running Container

```bash
docker ps
```

Expected:

```bash
0.0.0.0:9090->80/tcp
```

---

# Step 24 — Login into New Container

```bash
docker exec -it nginx-new-container bash
```

---

# Step 25 — Validate Website Data Inside Container

```bash
ls -l /usr/share/nginx/html/
```

Expected:

* Website files should be present

---

# Step 26 — Exit Container

```bash
exit
```

---

# Step 27 — Access Website from Browser

Open browser:

```text
http://<SERVER-IP>:9090
```

Expected:

* Same custom website should load
* Data persisted using Docker volume

---

# Scenario 2 — MySQL Container with Docker Volume Persistence

## Objective

* Create MySQL persistent storage
* Store database data in Docker volume
* Delete container and recover data
* Create new container using same volume

---

# Step 1 — Create MySQL Volume

```bash
docker volume create mysql-data
```

---

# Step 2 — Verify Volume

```bash
docker volume ls
```

---

# Step 3 — Run MySQL Container with Volume and Port Mapping

```bash
docker run -d \
--name mysql-container \
-p 3306:3306 \
-v mysql-data:/var/lib/mysql \
-e MYSQL_ROOT_PASSWORD=redhat \
mysql:8.0
```

Explanation:

* `/var/lib/mysql` stores MySQL database files
* Docker volume provides persistence

---

# Step 4 — Verify Running Container

```bash
docker ps
```

---

# Step 5 — Login into MySQL Container

```bash
docker exec -it mysql-container bash
```

---

# Step 6 — Login into MySQL Database

```bash
mysql -u root -p
```

Password:

```text
redhat
```

---

# Step 7 — Create Database

```sql
CREATE DATABASE companydb;
```

---

# Step 8 — Use Database

```sql
USE companydb;
```

---

# Step 9 — Create Table

```sql
CREATE TABLE employees (
id INT PRIMARY KEY AUTO_INCREMENT,
name VARCHAR(50),
department VARCHAR(50)
);
```

---

# Step 10 — Insert Records

```sql
INSERT INTO employees(name, department)
VALUES
('John','Linux'),
('David','DevOps'),
('Sam','Cloud');
```

---

# Step 11 — Verify Data

```sql
SELECT * FROM employees;
```

Expected:

```text
+----+-------+------------+
| id | name  | department |
+----+-------+------------+
|  1 | John  | Linux      |
|  2 | David | DevOps     |
|  3 | Sam   | Cloud      |
+----+-------+------------+
```

---

# Step 12 — Exit MySQL

```sql
exit
```

---

# Step 13 — Exit Container

```bash
exit
```

---

# Step 14 — Verify Volume Data on Host

```bash
ls -l /var/lib/docker/volumes/mysql-data/_data
```

Expected:

* MySQL database files should exist

---

# Step 15 — Stop MySQL Container

```bash
docker stop mysql-container
```

---

# Step 16 — Remove MySQL Container

```bash
docker rm mysql-container
```

---

# Step 17 — Verify Volume Still Exists

```bash
docker volume ls
```

Expected:

* `mysql-data` volume should still exist

---

# Step 18 — Create New MySQL Container with Same Volume and New Name

```bash
docker run -d \
--name mysql-new-container \
-p 3307:3306 \
-v mysql-data:/var/lib/mysql \
-e MYSQL_ROOT_PASSWORD=redhat \
mysql:8.0
```

---

# Step 19 — Verify Running Container

```bash
docker ps
```

Expected:

```bash
0.0.0.0:3307->3306/tcp
```

---

# Step 20 — Login into New MySQL Container

```bash
docker exec -it mysql-new-container bash
```

---

# Step 21 — Login into MySQL

```bash
mysql -u root -p
```

Password:

```text
redhat
```

---

# Step 22 — Verify Existing Database

```sql
SHOW DATABASES;
```

Expected:

* `companydb` should exist

---

# Step 23 — Use Database

```sql
USE companydb;
```

---

# Step 24 — Verify Existing Table Data

```sql
SELECT * FROM employees;
```

Expected:

* Previously inserted records should exist

---

# Step 25 — Exit MySQL and Container

```sql
exit
```

```bash
exit
```

---

# Browser Validation for Nginx

```text
http://<SERVER-IP>:8080
http://<SERVER-IP>:9090
```

---

# Useful Validation Commands

## Check Volumes

```bash
docker volume ls
```

---

## Inspect Container

```bash
docker inspect nginx-new-container
```

---

## Check Mounted Volumes

```bash
docker inspect mysql-new-container
```

---

## Check Logs

```bash
docker logs nginx-new-container
```

```bash
docker logs mysql-new-container
```

---

# Cleanup Exercise

## Stop Containers

```bash
docker stop nginx-new-container
docker stop mysql-new-container
```

---

## Remove Containers

```bash
docker rm nginx-new-container
docker rm mysql-new-container
```

---

## Remove Volumes

```bash
docker volume rm mydata
docker volume rm mysql-data
```

---

# Interview Questions

1. What is Docker volume persistence?
2. Why use Docker volumes for databases?
3. Difference between bind mount and named volume?
4. What happens to volume after container deletion?
5. Can multiple containers use same volume?
6. Why is persistent storage important for MySQL?
7. Where are Docker volumes stored in Linux?
8. How to inspect Docker volumes?
9. How to backup Docker volumes?
10. What happens if MySQL runs without volume mapping?
