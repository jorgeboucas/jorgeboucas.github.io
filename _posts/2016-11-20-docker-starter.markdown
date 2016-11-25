---
published: true
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
docker run --name <container name> -t -i <image tag> /bin/bash

# stop a container
docker stop <container name OR ID>

# start a containter
docker start <container name OR ID>

# start a container with a mapped host to guest folder and port mapping
docker run -p <host port>:<guest container port> -v <path to host folder>:<path to guest folder on container> --name <container name> -t -i <image tag> /bin/bash

# run a container in the background 
docker run -d --name <container name> -t -i <image tag>

# connect/execute a commmand on a running container
docker exec -i -t <container name> /bin/bash

# stop all containers
docker stop $(docker ps -q)

# remove all stopped containers
docker rm $(docker ps -a -q)

# remove all images
docker rmi $(docker images -q)

# remove MAC docker data 
rm -rf ~/Library/Containers/com.docker.docker/Data/*
```
