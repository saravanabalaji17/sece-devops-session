
### Lab Objective

* Build Docker image
* Push image to Docker Hub
* Pull image back
* Run container
* Validate Nginx website
* Stop and remove container

### Repository Structure

```text
webapp/
├── Dockerfile
└── .gitlab-ci.yml
```

### Dockerfile

```dockerfile
FROM ubuntu:24.04

LABEL maintainer="admin@localhost"

RUN apt update && \
    apt install -y nginx wget curl zip unzip && \
    rm -rf /var/lib/apt/lists/*

RUN wget -O /tmp/web.zip https://templatemo.com/download/templatemo_604_christmas_piano && \
    unzip -q /tmp/web.zip -d /tmp/web && \
    rm -rf /var/www/html/* && \
    cp -r /tmp/web/*/* /var/www/html/ && \
    rm -rf /tmp/web /tmp/web.zip

EXPOSE 80

CMD ["nginx", "-g", "daemon off;"]
```

### .gitlab-ci.yml

```yaml
stages:
  - build
  - test

build-image:
  stage: build

  image: docker:latest

  services:
    - docker:dind

  variables:
    DOCKER_TLS_CERTDIR: ""

  before_script:
    - docker login -u mydockeruser -p MyDockerPassword

  script:
    - docker build -t mydockeruser/christmas-piano:latest .
    - docker push mydockeruser/christmas-piano:latest

test-container:
  stage: test

  image: docker:latest

  services:
    - docker:dind

  variables:
    DOCKER_TLS_CERTDIR: ""

  before_script:
    - docker login -u mydockeruser -p MyDockerPassword

  script:
    - docker pull mydockeruser/christmas-piano:latest

    - docker run -d --name webtest -p 8080:80 mydockeruser/christmas-piano:latest

    - sleep 15

    - docker exec webtest curl -I http://localhost

    - docker logs webtest

    - docker stop webtest

    - docker rm webtest
```

### Expected Pipeline Flow

#### Stage 1: Build

```bash
docker build -t mydockeruser/christmas-piano:latest .
docker push mydockeruser/christmas-piano:latest
```

#### Stage 2: Test

```bash
docker pull mydockeruser/christmas-piano:latest

docker run -d --name webtest -p 8080:80 mydockeruser/christmas-piano:latest

curl http://localhost:8080

docker logs webtest

docker stop webtest
docker rm webtest
```

### Validation Commands on Runner Host

```bash
docker images

docker ps

docker pull mydockeruser/christmas-piano:latest

docker run -d -p 8080:80 mydockeruser/christmas-piano:latest

curl http://localhost:8080
```

#
