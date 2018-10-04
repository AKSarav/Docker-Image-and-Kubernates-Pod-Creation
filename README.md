# Docker-Image-and-Kubernates-Pod-Creation
Steps to create a Docker Image and upload to Docker hub and Creating a New Kubernates POD using the same image using Kubernates Manifest file

### Step1:
*Creating of Dockerfile. The file is designed to run redis in-memory database in an alpine base OS*
```Go
# Use existing docker image as a base
FROM alpine

# Download and install dependency
RUN apk add --update redis

# EXPOSE the port to the Host OS
EXPOSE 6379

# Tell the image what command it has to execute as it starts as a container
CMD ["redis-server"]
```

### Step2:
*Build the Image using the Dockerfile we have developed*
```bash
aksarav@middlewareinventory:/apps/docker/redisserver$ docker build -t saravak/redis .
Sending build context to Docker daemon  2.048kB
Step 1/4 : FROM alpine
 ---> 196d12cf6ab1
Step 2/4 : RUN apk add --update redis
 ---> Using cache
 ---> a1426a22089a
Step 3/4 : EXPOSE 6379
 ---> Using cache
 ---> 7c0fde02a01c
Step 4/4 : CMD ["redis-server"]
 ---> Using cache
 ---> 8e1cc8b503d8
Successfully built 8e1cc8b503d8
Successfully tagged saravak/redis:latest
aksarav@middlewareinventory:/apps/docker/redisserver$ 
```

### Step3:
*Make sure the image is ready and listing in the docker images list*
```bash
aksarav@middlewareinventory:/apps/docker/redisserver$ docker images
REPOSITORY          TAG                 IMAGE ID            CREATED             SIZE
saravak/redis       latest              8e1cc8b503d8        9 hours ago         6.9MB
redis               latest              0a153379a539        45 hours ago        83.4MB
busybox             latest              59788edf1f3e        46 hours ago        1.15MB
tomcat              latest              41a54fe1f79d        3 weeks ago         463MB
alpine              latest              196d12cf6ab1        3 weeks ago         4.41MB
```

### Step4
*Upload the image to the hub.docker.com repository for global access*
```bash
aksarav@middlewareinventory:/apps/docker/redisserver$ docker login
Login with your Docker ID to push and pull images from Docker Hub. If you don't have a Docker ID, head over to https://hub.docker.com to create one.
Username: saravak
Password: 
Login Succeeded

aksarav@middlewareinventory:/apps/docker/redisserver$ docker push saravak/redis
The push refers to repository [docker.io/saravak/redis]
a63649d27e03: Layer already exists 
df64d3292fd6: Layer already exists 
latest: digest: sha256:dc0631a78737b5f0be09ad4c27b0120c916feb06d9bd7ce1fd6890925f5dd42b size: 739
aksarav@middlewareinventory:/apps/docker/redisserver$ 
```

