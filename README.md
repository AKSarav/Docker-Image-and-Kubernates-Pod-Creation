# Docker-Image-and-Kubernates-Pod-Creation
Steps to create a Docker Image and upload to Docker hub and Creating a New Kubernates POD using the same image using Kubernates Manifest file

## Docker Tasks

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
```yaml
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

### Step5
*Start the container using the Image we just built*
```java
aksarav@middlewareinventory:/apps/docker/redisserver$ docker container run -d -it --name rediscontainer saravak/redis:latest 
b9824eb84fd75fdf511149284db8fef4b1d03dce6be5e8527e38159b672f115c
aksarav@middlewareinventory:/apps/docker/redisserver$ docker container list
CONTAINER ID        IMAGE                  COMMAND             CREATED             STATUS              PORTS               NAMES
b9824eb84fd7        saravak/redis:latest   "redis-server"      27 seconds ago      Up 25 seconds       6379/tcp            rediscontainer
```


## Kubernates Tasks

### Step6
*Create a Manifest file to create a Simple and Straight forward POD [Without replica and Scaling]*
```yaml
apiVersion: v1
kind: Pod
metadata:
   name: radis-pod
spec:
   containers:
   - name: redis-container01
     image: saravak/redis:latest
     ports:
     - containerPort: 6379
```

### Step7
*Create a POD using Kubectl command using the Manifest file we have created in Step6*
```java
aksarav@middlewareinventory:/apps/kubernetes$ kubectl create -f create-redispod.yml
pod/radis-pod created
aksarav@middlewareinventory:/apps/kubernetes$ kubectl get pods
NAME                              READY   STATUS    RESTARTS   AGE
hello-minikube-7c77b68cff-pd4x2   1/1     Running   1          11h
radis-pod                         1/1     Running   0          17s
```

### Step8
*Get the status and more detailed information on the newly created POD*
```java
aksarav@middlewareinventory:/apps/kubernetes$ kubectl get pods
NAME                              READY   STATUS    RESTARTS   AGE
hello-minikube-7c77b68cff-pd4x2   1/1     Running   1          11h
radis-pod                         1/1     Running   0          2m
aksarav@middlewareinventory:/apps/kubernetes$ kubectl get pods/radis-pod
NAME        READY   STATUS    RESTARTS   AGE
radis-pod   1/1     Running   0          2m
aksarav@middlewareinventory:/apps/kubernetes$ kubectl describe pods/radis-pod
Name:         radis-pod
Namespace:    default
Node:         minikube/192.168.64.2
Start Time:   Thu, 04 Oct 2018 21:58:28 +0530
Labels:       <none>
Annotations:  <none>
Status:       Running
IP:           172.17.0.6
Containers:
  redis-container01:
    Container ID:   docker://c7bc7ce68272493477249da617ea042ca5191b6b7b4ef89f9490dab8584e0fb4
    Image:          saravak/redis:latest
    Image ID:       docker-pullable://saravak/redis@sha256:dc0631a78737b5f0be09ad4c27b0120c916feb06d9bd7ce1fd6890925f5dd42b
    Port:           6379/TCP
    Host Port:      0/TCP
    State:          Running
      Started:      Thu, 04 Oct 2018 21:58:36 +0530
    Ready:          True
    Restart Count:  0
    Environment:    <none>
    Mounts:
      /var/run/secrets/kubernetes.io/serviceaccount from default-token-t5c7w (ro)
Conditions:
  Type           Status
  Initialized    True 
  Ready          True 
  PodScheduled   True 
Volumes:
  default-token-t5c7w:
    Type:        Secret (a volume populated by a Secret)
    SecretName:  default-token-t5c7w
    Optional:    false
QoS Class:       BestEffort
Node-Selectors:  <none>
Tolerations:     node.kubernetes.io/not-ready:NoExecute for 300s
                 node.kubernetes.io/unreachable:NoExecute for 300s
Events:
  Type    Reason                 Age    From               Message
  ----    ------                 ----   ----               -------
  Normal  Scheduled              2m27s  default-scheduler  Successfully assigned radis-pod to minikube
  Normal  SuccessfulMountVolume  2m27s  kubelet, minikube  MountVolume.SetUp succeeded for volume "default-token-t5c7w"
  Normal  Pulling                2m26s  kubelet, minikube  pulling image "saravak/redis:latest"
  Normal  Pulled                 2m20s  kubelet, minikube  Successfully pulled image "saravak/redis:latest"
  Normal  Created                2m19s  kubelet, minikube  Created container
  Normal  Started                2m19s  kubelet, minikube  Started container
aksarav@middlewareinventory:/apps/kubernetes$ 

```

### Step9
_Create a Manifest file with **Replication Controller** to acheive **desired state** with following content and **create** it_ 

```
apiVersion: v1
kind: ReplicationController
metadata:
  name: redis-db-desiredstate-rc
spec:
  replicas: 5
  selector:
    app: redis-app
  template:
    metadata:
      labels:
        app: redis-app
        zone: production
    spec:
      containers:
      - name: redispod
        image: saravak/redis:latest
        ports:
        - containerPort: 6379
```
```
aksarav@middlewareinventory:/apps/kubernetes$ kubectl create -f create-redispod-scalable-replicationcontrollers.yml 
replicationcontroller/redis-db-desiredstate-rc created
```

### Step10
_check newly created **replicationcontroller** status and see **the statistics** shown_

```
aksarav@middlewareinventory:/apps/kubernetes$ kubectl get replicationcontroller
NAME                       DESIRED   CURRENT   READY   AGE
redis-db-desiredstate-rc   5         5         0       6s
```

_now run ```kubectl get pods``` to see the newly created pods with replication controller_
```
aksarav@middlewareinventory:/apps/kubernetes$ kubectl get pods
NAME                             READY   STATUS    RESTARTS   AGE
nginx-65899c769f-xvcmr           1/1     Running   1          5d
radis-pod                        2/2     Running   0          14m
redis-db-desiredstate-rc-7qdjg   1/1     Running   0          35s
redis-db-desiredstate-rc-l5df4   1/1     Running   0          35s
redis-db-desiredstate-rc-s4pvr   1/1     Running   0          35s
redis-db-desiredstate-rc-v9kns   1/1     Running   0          35s
redis-db-desiredstate-rc-zfwd2   1/1     Running   0          35s
```

### Step11
_Let us test if we forcefully remove or delete a one of the pod from this **desired state pool** if K8s recreate it or not_

Deleting one of the pod

```
aksarav@middlewareinventory:/apps/kubernetes$ kubectl delete pod redis-db-desiredstate-rc-7qdjg
pod "redis-db-desiredstate-rc-7qdjg" deleted
```

check the replication controller status and get pods list to verify
```
aksarav@middlewareinventory:/apps/kubernetes$ kubectl get replicationcontroller
NAME                       DESIRED   CURRENT   READY   AGE
redis-db-desiredstate-rc   5         5         4       7m
aksarav@middlewareinventory:/apps/kubernetes$ kubectl get pods
NAME                             READY   STATUS    RESTARTS   AGE
nginx-65899c769f-xvcmr           1/1     Running   1          5d
radis-pod                        2/2     Running   0          21m
redis-db-desiredstate-rc-l5df4   1/1     Running   0          7m
redis-db-desiredstate-rc-lllbh   1/1     Running   0          11s
redis-db-desiredstate-rc-s4pvr   1/1     Running   0          7m
redis-db-desiredstate-rc-v9kns   1/1     Running   0          7m
redis-db-desiredstate-rc-zfwd2   1/1     Running   0          7m
```

_**MAGIC** you can see K8s have already replaced that deleted pod with a new pod named ```redis-db-desiredstate-rc-lllbh```. You can check the AGE having **11 seconds** which proves this is new_




