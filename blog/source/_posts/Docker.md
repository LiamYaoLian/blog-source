---
title: Docker
date: 2022-12-07 13:36:45
tags:
---
```

docker pull ubuntu:18.04
docker run -it ubuntu:18.04 sh

# commands below will be executed in the container
cat /etc/os-release
apt update
apt install -y wget redis
redis-server &
```
`docker pull` to pull images from remote registry
`docker images` to list local images
`docker rmi` to remove local images
`-d`
`-it`
