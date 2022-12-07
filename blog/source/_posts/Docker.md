---
title: Docker
date: 2022-12-07 13:36:45
tags:
---
# Client Commands
* `docker pull` to pull images from remote registry: `docker pull ubuntu:18.04`
* `docker images` to list local images
* `docker rmi` to remove image
* `docker run` to run a container
  - `-d`: run in the background
  - `-it`: `docker run -it ubuntu:18.04 sh` will enter this container
  - `--rm`: remove after the container stops
  - `--name`: give the container a name

* `docker exec` will not create a new container: `docker exec -it red_srv sh`
* `docker ps`
  - `-a` means all
* `docker stop` to stop a container
* `docker rm` to remove a container

# Dockerfile
To build an image
```
# Dockerfile.busybox

# chose a base image
FROM busybox

# run this command when container starts            
CMD echo "hello world"  
```
# docker build
```
docker build -f Dockerfile.busybox .
```
It will set context. Files in the context will be uploaded to Docker daemon. Do not put unnecessary files in the context or name them in `.dockerignore`.

```
# Copy a.txt in the context to /tmp in the image
COPY ./a.txt  /tmp/a.txt

# Error: cannot copy file not in the context
COPY /etc/hosts  /tmp
```
* run
```
RUN apt-get update \
    && apt-get install -y \
        build-essential \
        curl \
        make \
        unzip \
    && cd /tmp \
    && curl -fSL xxx.tar.gz -o xxx.tar.gz\
    && tar xzf xxx.tar.gz \
    && cd xxx \
    && ./config \
    && make \
    && make clean
```
Alternative way:
```
# Copy shell script to /tmp
COPY setup.sh  /tmp/                

# add permission
# run and delete shell script
RUN cd /tmp && chmod +x setup.sh \  
    && ./setup.sh && rm setup.sh    
```
* variable
- the scope of variable created by ARG is within building image
- the scope of variable created by ENV is in both building and running image
```
ARG IMAGE_BASE="node"
ARG IMAGE_TAG="alpine"

ENV PATH=$PATH:/tmp
ENV DEBUG=OFF
```
* expose
```
EXPOSE 443           # default protocol: tcp
EXPOSE 53/udp        # set to udp protocol
```

# Java Spring Boot + Docker
https://docs.docker.com/language/java/

https://stackoverflow.com/questions/63851313/connection-refused-in-docker-container-between-database-container-and-java-conta

https://stackoverflow.com/questions/50379839/connection-java-mysql-public-key-retrieval-is-not-allowed

* volume will not be deleted after container is deleted
