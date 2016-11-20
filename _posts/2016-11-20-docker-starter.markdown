---
published: false
title: docker starter
layout: post
---

```bash
# build image from Dockerfile
docker build -t <image tag> /path/to/folder/conataining/the/Dockerfile/

# or 
cd /path/to/folder/conataining/the/Dockerfile/
docker build -t <image tag> .

# run an image and start an interactive bash shell
docker run -t -i <image tag> /bin/bash ## keep on here with --name

# stop all containers
docker stop $(docker ps -q)

# remove all stopped containers
docker rm $(docker ps -a -q)

# remove all images
docker rmi $(docker images -q)

# remove MAC docker data 
rm -rf ~/Library/Containers/com.docker.docker/Data/*
```
