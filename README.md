# ULL-DevOps-Docker-Swarm

## Environment Set Up

192.168.20.20 --> Ansible Control Server (1)
192.168.20.30 --> Swarm Manager Server  (2)

1. Playbook docker.yml 

![Docker](/images/1.png)

2. Add new IP in /etc/ansible/hosts 

```
[swarm_manager]
192.168.20.27 ansible_ssh_user=usuario
```

3. ssh-copy-id usuario@192.168.20.27

```
usuario@192.168.20.27's password: 

Number of key(s) added: 1

Now try logging into the machine, with:   "ssh 'usuario@192.168.20.27'"
and check to make sure that only the key(s) you wanted were added.
```

4. Add new ansible role 

```
ansible-galaxy init docker
```

```
[control@aitic_grupo1_ansible_control tasks]$ cat main.yml 
---
# tasks file for docker

    
    - name: Update apt-get repo and cache
      apt: update_cache=yes force_apt_get=yes cache_valid_time=3600

    - name: Install preconditions
      shell: apt-get install -y \
                apt-transport-https -y \
                ca-certificates \
                curl \
                gnupg-agent \
                software-properties-common
    - name: Add Docker’s official GPG key
      shell: curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo apt-key add -

    - name: Install Docker CE
      apt:
        name: ['docker-ce', 'docker-ce-cli', 'containerd.io']
    - service: name=docker state=restarted
```
![Docker](/images/2.png)

5. Install Docker on (2) and check the installed version
```
# docker version
Client: Docker Engine - Community
 Version:           19.03.12
 API version:       1.40
 Go version:        go1.13.10
 Git commit:        48a66213fe
 Built:             Mon Jun 22 15:45:36 2020
 OS/Arch:           linux/amd64
 Experimental:      false

Server: Docker Engine - Community
 Engine:
  Version:          19.03.12
  API version:      1.40 (minimum version 1.12)
  Go version:       go1.13.10
  Git commit:       48a66213fe
  Built:            Mon Jun 22 15:44:07 2020
  OS/Arch:          linux/amd64
  Experimental:     false
 containerd:
  Version:          1.2.13
  GitCommit:        7ad184331fa3e55e52b890ea95e65ba581ae3429
 runc:
  Version:          1.0.0-rc10
  GitCommit:        dc9208a3303feef5b3839f4323d9beb36df0a9dd
 docker-init:
  Version:          0.18.0
  GitCommit:        fec3683
```

6. Create Swarm_Worker_1 VM using ansible playbook
IP --> 192.168.20.31 (3)

7. Install docker on (3) and check version

![Docker](/images/3.png)

```
# docker version
Client: Docker Engine - Community
 Version:           19.03.12
 API version:       1.40
 Go version:        go1.13.10
 Git commit:        48a66213fe
 Built:             Mon Jun 22 15:45:36 2020
 OS/Arch:           linux/amd64
 Experimental:      false

Server: Docker Engine - Community
 Engine:
  Version:          19.03.12
  API version:      1.40 (minimum version 1.12)
  Go version:       go1.13.10
  Git commit:       48a66213fe
  Built:            Mon Jun 22 15:44:07 2020
  OS/Arch:          linux/amd64
  Experimental:     false
 containerd:
  Version:          1.2.13
  GitCommit:        7ad184331fa3e55e52b890ea95e65ba581ae3429
 runc:
  Version:          1.0.0-rc10
  GitCommit:        dc9208a3303feef5b3839f4323d9beb36df0a9dd
 docker-init:
  Version:          0.18.0
  GitCommit:        fec3683
```

8. Create Swarm_Worker_2 VM using ansible playbook
IP --> 192.168.20.32 (4)

9. Install docker on (3) and check version
![Docker](/images/3.png)

```
# docker version
Client: Docker Engine - Community
 Version:           19.03.12
 API version:       1.40
 Go version:        go1.13.10
 Git commit:        48a66213fe
 Built:             Mon Jun 22 15:45:36 2020
 OS/Arch:           linux/amd64
 Experimental:      false

Server: Docker Engine - Community
 Engine:
  Version:          19.03.12
  API version:      1.40 (minimum version 1.12)
  Go version:       go1.13.10
  Git commit:       48a66213fe
  Built:            Mon Jun 22 15:44:07 2020
  OS/Arch:          linux/amd64
  Experimental:     false
 containerd:
  Version:          1.2.13
  GitCommit:        7ad184331fa3e55e52b890ea95e65ba581ae3429
 runc:
  Version:          1.0.0-rc10
  GitCommit:        dc9208a3303feef5b3839f4323d9beb36df0a9dd
 docker-init:
  Version:          0.18.0
  GitCommit:        fec3683
  ```

10. Open protocols and ports between the hosts

On each node that will function as a manager/worker, execute the following commands:

```
ufw allow 2377/tcp
ufw allow 7946/tcp
ufw allow 7946/udp
ufw allow 4789/udp

ufw enable

systemctl restart docker
```

Ya están abiertos --> no hace falta

## Create Swarm

Install Docker Machine (opt)

```
# base=https://github.com/docker/machine/releases/download/v0.16.0 &&
>   curl -L $base/docker-machine-$(uname -s)-$(uname -m) >/tmp/docker-machine &&
>   sudo mv /tmp/docker-machine /usr/local/bin/docker-machine &&
>   chmod +x /usr/local/bin/docker-machine
```

```
# docker-machine version
docker-machine version 0.16.0, build 702c267f
```

1. Create Swarm Manager

```
docker swarm init --advertise-addr 192.168.20.27
```
```
Swarm initialized: current node (21zfgkavepn2z349ac3m31o3j) is now a manager.

To add a worker to this swarm, run the following command:

    docker swarm join --token SWMTKN-1-5zgbvamgc5zcnib4l5ablpysjirv12r4ycaat50q4ib9d01me4-4sehycdyr9b9b7in3gsskdllk 192.168.20.27:2377

To add a manager to this swarm, run 'docker swarm join-token manager' and follow the instructions.
```

Note: for initiating the same commant type
```
docker swarm join-token manager
```

For workers:
```
docker swarm join-token worker
```

Check current state 
```
# docker node ls
ID                            HOSTNAME            STATUS              AVAILABILITY        MANAGER STATUS      ENGINE VERSION
21zfgkavepn2z349ac3m31o3j *   ubuntu              Ready               Active              Leader              19.03.12
```

2. Add workers on (2), (3)

```
# docker swarm join --token SWMTKN-1-5zgbvamgc5zcnib4l5ablpysjirv12r4ycaat50q4ib9d01me4-4sehycdyr9b9b7in3gsskdllk 192.168.20.27:2377
This node joined a swarm as a worker.
```
Check current state 
```
ID                            HOSTNAME            STATUS              AVAILABILITY        MANAGER STATUS      ENGINE VERSION
l5ztv11jfkyjl4ifouny8wgth *   manager             Ready               Active              Leader              19.03.12
pvq9579y790c50y9dmcilceot     worker1             Ready               Active                                  19.03.12
wfiteggh59zid2njam23jxzdn     worker2             Ready               Active                                  19.03.12
```
Change hostname 

```
hostname manager
service docker restart
```


## Deploy a service to the swarm

1. Run the following command on manager machine: 
```
# docker service create --replicas 3 --name helloworld alpine ping docker.com
wssym5qnk76at52e34sv5ualx
overall progress: 3 out of 3 tasks 
1/3: running   [==================================================>] 
2/3: running   [==================================================>] 
3/3: running   [==================================================>]
```
```
# docker service ls
ID                  NAME                MODE                REPLICAS            IMAGE                 PORTS
wssym5qnk76a        helloworld          replicated          3/3                 alpine:latest       
```


## Deploy Realworld App with *docker-compose*

1. Clone Realworld App from github --> https://github.com/gothinkster/node-express-realworld-example-app.git
2. Add Dockerfile
```
FROM node:12
  
# Create app directory
WORKDIR /usr/src/app

# Install app dependencies
# A wildcard is used to ensure both package.json AND package-lock.json are copied
# where available (npm@5+)
COPY package*.json ./

RUN npm install
# If you are building your code for production
# RUN npm ci --only=production

# Bundle app source
COPY . .

EXPOSE 8080
CMD [ "node", "app.js" ]
```
3. Install docker-compose
4. Add docker-compose.yml

```
version: '3'
services:
  web:
    image: node-web-app
    build: .
    command: "node app.js"
    ports:
      - "3000:3000"
    depends_on:
      - "mongo"
  mongo:
    image: "mongo"
    ports:
      - "27017:27017"
```
5. docker-compose up --build -d mongo
6. docker-compose up --build -d web
7. docker-compose up 

![Docker](/images/8.png)

## Deploy Realworld App to the swarm

1. Clone Realworld App from github --> https://github.com/gothinkster/react-redux-realworld-example-app.git
2. Add Dockerfile

```
FROM node:12
  
# Create app directory
WORKDIR /usr/src/app

# Install app dependencies
# A wildcard is used to ensure both package.json AND package-lock.json are copied
# where available (npm@5+)
COPY package*.json ./

RUN npm install
# If you are building your code for production
# RUN npm ci --only=production

# Bundle app source
COPY . .

EXPOSE 4100
CMD [ "npm", "start" ]
```

3. Build image 
```
docker build -t web-app .
```
```
# docker images
REPOSITORY          TAG                 IMAGE ID            CREATED             SIZE
web-app             latest              91453a6c9676        11 hours ago        1.24GB
```

4. Create replicated service
```
docker service create -p 3000:4100 --replicas 3 --name realworld web-app
```

5. Check results
```
# docker service ls
ID                  NAME                MODE                REPLICAS            IMAGE               PORTS
wssym5qnk76a        helloworld          replicated          3/3                 alpine:latest       
y56fkvq2jffr        mongodb             replicated          1/1                 mongo:latest        
b7yz7az78xyq        realworld           replicated          3/3                 web-app:latest      *:3000->4100/tcp
````

Swarm Manager
```
curl http://192.168.20.30:3000/
curl http://192.168.20.31:3000/
curl http://192.168.20.32:3000/
````

![Docker](/images/9.png)

Worker_1

![Docker](/images/10.png)

Worker_2

![Docker](/images/11.png)


## Kubernetes

### Preconditions 

+ Install

```
yum install python3
```
```
yum install python36-devel
```
```
easy_install-3.6 pip
```
```
yum install python-netaddr
```
```
yum install python-pip
```
```
pip install -U Jinja2
```
```

```

+ Configure NOPASSWD for sudoers on the cluster machines 
```
sudo visudo
```
Add the folowwing:
```
root     ALL=(ALL) NOPASSWD:ALL
```

```
[root@new_control ansible]# pip3 -V
pip 9.0.3 from /usr/lib/python3.6/site-packages (python 3.6)
```

### Kubernetes

```
git clone https://github.com/kubernetes-sigs/kubespray.git
```

```
[root@new_control ansible]# cd kubespray/
[root@new_control kubespray]# pip3 install -r requirements.txt
```
```
[root@new_control kubespray]# cp -rfp inventory/sample inventory/mycluster
[root@new_control kubespray]# declare -a IPS=(192.168.20.36 192.168.20.37 192.168.20.34 192.168.20.35)
```

```
CONFIG_FILE=inventory/mycluster/hosts.yml python3 contrib/inventory_builder/inventory.py ${IPS[@]}
```
Result

```
all:
  hosts:
    node1:
      ansible_host: 192.168.20.36
      ip: 192.168.20.36
      access_ip: 192.168.20.36
    node2:
      ansible_host: 192.168.20.37
      ip: 192.168.20.37
      access_ip: 192.168.20.37
    node3:
      ansible_host: 192.168.20.34
      ip: 192.168.20.34
      access_ip: 192.168.20.34
    node4:
      ansible_host: 192.168.20.35
      ip: 192.168.20.35
      access_ip: 192.168.20.35
  children:
    kube-master:
      hosts:
        node1:
        node2:
    kube-node:
      hosts:
        node1:
        node2:
        node3:
        node4:
    etcd:
      hosts:
        node1:
        node2:
        node3:
    k8s-cluster:
      children:
        kube-master:
        kube-node:
    calico-rr:
      hosts: {}
```

Cluster related configurations
```
cat inventory/mycluster/group_vars/k8s-cluster/k8s-cluster.yml
```
**kube_version: v1.18.5**

## Run Ansible Playbook

```
ansible-playbook -i inventory/mycluster/hosts.yml --user usuario cluster.yml -K
```
```
root@node2:/etc/kubernetes# kubectl get nodes
NAME    STATUS   ROLES    AGE     VERSION
node1   Ready    master   8m17s   v1.18.5
node2   Ready    master   6m12s   v1.18.5
node3   Ready    <none>   4m30s   v1.18.5
node4   Ready    <none>   4m29s   v1.18.5
root@node2:/etc/kubernetes# kubectl version --short
Client Version: v1.18.5
Server Version: v1.18.5
root@node2:/etc/kubernetes# kubectl cluster-info
Kubernetes master is running at https://192.168.20.36:6443

To further debug and diagnose cluster problems, use 'kubectl cluster-info dump'.
root@node2:/etc/kubernetes# kubectl -n kube-system get pods
NAME                                          READY   STATUS    RESTARTS   AGE
NAME                                          READY   STATUS    RESTARTS   AGE
calico-kube-controllers-6b649f597-q7bwj       1/1     Running   0          3m40s
calico-node-652pk                             1/1     Running   1          4m43s
calico-node-6cd77                             1/1     Running   1          4m43s
calico-node-htv2l                             1/1     Running   1          4m43s
calico-node-l6gx8                             1/1     Running   1          4m43s
coredns-dff8fc7d-992tb                        1/1     Running   0          3m4s
coredns-dff8fc7d-wnc7b                        1/1     Running   0          2m47s
dns-autoscaler-66498f5c5f-p2sdz               1/1     Running   0          2m54s
kube-apiserver-node1                          1/1     Running   0          8m49s
kube-apiserver-node2                          1/1     Running   0          6m54s
kube-controller-manager-node1                 1/1     Running   0          8m49s
kube-controller-manager-node2                 1/1     Running   0          6m54s
kube-proxy-7tzcf                              1/1     Running   0          5m23s
kube-proxy-8djmq                              1/1     Running   0          5m23s
kube-proxy-rqsrw                              1/1     Running   0          5m21s
kube-proxy-w9lz4                              1/1     Running   0          5m20s
kube-scheduler-node1                          1/1     Running   0          8m49s
kube-scheduler-node2                          1/1     Running   0          6m54s
kubernetes-dashboard-57777fbdcb-8wzcj         1/1     Running   0          2m42s
kubernetes-metrics-scraper-54fbb4d595-vfrm6   1/1     Running   0          2m41s
nginx-proxy-node3                             1/1     Running   0          5m28s
nginx-proxy-node4                             1/1     Running   0          5m27s
nodelocaldns-chmds                            1/1     Running   0          2m48s
nodelocaldns-jxhmw                            1/1     Running   0          2m48s
nodelocaldns-whskr                            1/1     Running   0          2m48s
nodelocaldns-zzz46                            1/1     Running   0          2m48s
root@node2:/etc/kubernetes# kubectl get pods
No resources found in default namespace.
```

![Docker](/images/12.png)

## Containers


Terminal 1

```
kubectl run myshell -it --rm --image busybox -- sh
```
```
root@node2:/etc/kubernetes# kubectl run myshell -it --rm --image busybox -- sh
If you don't see a command prompt, try pressing enter.
/ # hostname -i
10.233.96.5
/ # ping 10.233.96.6
PING 10.233.96.6 (10.233.96.6): 56 data bytes
64 bytes from 10.233.96.6: seq=0 ttl=63 time=0.579 ms
64 bytes from 10.233.96.6: seq=1 ttl=63 time=0.096 ms
```


Terminal 2
```
kubectl run myshell2 -it --rm --image busybox -- sh
```
```
root@node2:/home/usuario# kubectl run myshell2 -it --rm --image busybox -- sh
If you don't see a command prompt, try pressing enter.
/ # hostname -i
10.233.96.6
/ # ping 10.233.96.5
PING 10.233.96.5 (10.233.96.5): 56 data bytes
64 bytes from 10.233.96.5: seq=0 ttl=63 time=0.240 ms
64 bytes from 10.233.96.5: seq=1 ttl=63 time=0.119 ms
```

## hello-kubernetes

*Master node2*

```
git clone https://github.com/paulbouwer/hello-kubernetes.git
```
```
root@node2:/etc/kubernetes/hello-kubernetes# kubectl apply -f yaml/hello-kubernetes.custom-message.yaml
service/hello-kubernetes-custom created
deployment.apps/hello-kubernetes-custom created
```

```
# kubectl get service hello-kubernetes
NAME               TYPE           CLUSTER-IP     EXTERNAL-IP   PORT(S)        AGE
hello-kubernetes   LoadBalancer   10.233.9.214   <pending>     80:31123/TCP   6m16s
```
```
root@node2:/etc/kubernetes/hello-kubernetes/app# ping 10.233.9.214
PING 10.233.9.214 (10.233.9.214) 56(84) bytes of data.
64 bytes from 10.233.9.214: icmp_seq=1 ttl=64 time=0.121 ms
```

![Docker](/images/13.png)

*Master node1*
```
root@node1:/etc# ping 10.233.9.214
PING 10.233.9.214 (10.233.9.214) 56(84) bytes of data.
64 bytes from 10.233.9.214: icmp_seq=1 ttl=64 time=0.162 ms
64 bytes from 10.233.9.214: icmp_seq=2 ttl=64 time=0.078 ms
64 bytes from 10.233.9.214: icmp_seq=3 ttl=64 time=0.096 ms
```
```
root@node2:/etc/kubernetes/hello-kubernetes/app# kubectl get pods -o wide
NAME                                       READY   STATUS    RESTARTS   AGE   IP             NODE    NOMINATED NODE   READINESS GATES
hello-kubernetes-594f6f475f-749x6          1/1     Running   0          43s   10.233.90.2    node1   <none>           <none>
hello-kubernetes-594f6f475f-c8zqd          1/1     Running   0          43s   10.233.92.2    node3   <none>           <none>
hello-kubernetes-594f6f475f-mvncc          1/1     Running   0          43s   10.233.105.4   node4   <none>           <none>
hello-kubernetes-custom-7555f78555-hxj5h   1/1     Running   0          3m    10.233.92.1    node3   <none>           <none>
hello-kubernetes-custom-7555f78555-xvbsg   1/1     Running   0          3m    10.233.96.3    node2   <none>           <none>
hello-kubernetes-custom-7555f78555-z8x4z   1/1     Running   0          3m    10.233.105.3   node4   <none>           <none>
```


## Realworld Frontend

```
git clone
```

Add Dockerfile
```
root@node1:/etc/kubernetes/react-redux-realworld-example-app# cat Dockerfile 
FROM node:13.6.0-alpine

# Create app directory
RUN mkdir -p /usr/src/app
WORKDIR /usr/src/app

# Install app dependencies
COPY package.json /usr/src/app/
RUN npm install

# Bundle app source
COPY . /usr/src/app

USER node
CMD [ "npm", "start" ]
```
Build image
```
docker build -t react-kubernetes .
```

Add .yaml file with **imagePullPolicy: Never** as we're using a local image
```
root@node1:/etc/kubernetes/react-redux-realworld-example-app# cat react-kubernetes.yaml 
apiVersion: v1
kind: Service
metadata:
  name: react-kubernetes
spec:
  type: LoadBalancer
  ports:
  - port: 80
    targetPort: 3000
  selector:
    app: react-kubernetes
---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: react-kubernetes
spec:
  replicas: 3
  selector:
    matchLabels:
      app: react-kubernetes
  template:
    metadata:
      labels:
        app: react-kubernetes
    spec:
      containers:
      - name: react-kubernetes
        image: react-kubernetes
        imagePullPolicy: Never
        ports:
        - containerPort: 3000
```

```
kubectl apply -f react-kubernetes.yaml
```

```
root@node1:/etc/kubernetes/react-redux-realworld-example-app# kubectl get pods
NAME                                       READY   STATUS              RESTARTS   AGE
hello-kubernetes-594f6f475f-749x6          1/1     Running             0          124m
hello-kubernetes-594f6f475f-c8zqd          1/1     Running             0          124m
hello-kubernetes-594f6f475f-mvncc          1/1     Running             0          124m
hello-kubernetes-custom-7555f78555-hxj5h   1/1     Running             0          127m
hello-kubernetes-custom-7555f78555-xvbsg   1/1     Running             0          127m
hello-kubernetes-custom-7555f78555-z8x4z   1/1     Running             0          127m
react-kubernetes-5c87479c9b-5zqtq          0/1     ErrImageNeverPull   0          8m12s
react-kubernetes-5c87479c9b-s7499          1/1     Running             0          12m
react-kubernetes-5db96cdc7c-2m4wk          0/1     ErrImageNeverPull   0          8m
react-kubernetes-f8bc75bc7-pp757           0/1     ImagePullBackOff    0          7m45s
```

Build on all nodes???????

## MongoDB

Add .yaml file
```
apiVersion: v1
kind: Service
metadata:
  name: mongo
  labels:
    run: mongo
spec:
  ports:
  - port: 27017
    targetPort: 27017
    protocol: TCP
  selector:
    run: mongo
---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: mongo
spec:
  replicas: 3
  selector:
    matchLabels:
      run: mongo
  template:
    metadata:
      labels:
        run: mongo
    spec:
      containers:
      - name: mongo
        image: mongo
        ports:
        - containerPort: 27017
```

```
kubectl apply -f mongo.yaml
```
```
root@node1:/etc/kubernetes/node-express-realworld-example-app# kubectl get pods -o wide
NAME                                       READY   STATUS              RESTARTS   AGE    IP              NODE     NOMINATED NODE   READINESS GATES
mongo-5bc9c9dbbf-kpvkv                     1/1     Running             0          4m9s   10.233.92.12    node3    <none>           <none>
mongo-5bc9c9dbbf-mmjf2                     1/1     Running             0          4m9s   10.233.90.9     node1    <none>           <none>
mongo-5bc9c9dbbf-nw9jk                     1/1     Running             0          4m9s   10.233.105.10   node4    <none>           <none>
```

## Realworld BackEnd

Add Dockerfile 

```
FROM node:13.6.0-alpine
  
# Create app directory
RUN mkdir -p /usr/src/app
WORKDIR /usr/src/app

# Install app dependencies
COPY package.json /usr/src/app/
RUN npm install

# Bundle app source
COPY . /usr/src/app

EXPOSE 3000
ENV PORT=3000
ENV MONGO_SERVICE_HOST=mongo
ENV MONGO_SERVICE_PORT=27017

USER node
CMD [ "npm", "start" ]
```

```
# docker build -t realworld .
```

```
root@node1:/etc/kubernetes/node-express-realworld-example-app# kubectl apply -f realworld.yaml
service/realworld created
deployment.apps/realworld created
```

```
root@node1:/etc/kubernetes/node-express-realworld-example-app# kubectl get pods -o wide
NAME                                       READY   STATUS              RESTARTS   AGE   IP              NODE     NOMINATED NODE   READINESS GATES
realworld-d8b559548-5cnr7                  1/1     Running             0          19s   10.233.90.10    node1    <none>           <none>
realworld-d8b559548-kph5g                  0/1     ErrImageNeverPull   0          96s   10.233.92.14    node3    <none>           <none>
realworld-d8b559548-rxmb8                  0/1     ErrImageNeverPull   0          96s   10.233.105.11   node4    <none>           <none>
realworld-d8b559548-xnkkc                  0/1     ErrImageNeverPull   0          96s   10.233.96.7     node2    <none>           <none>
```


## Push containers to docker hub

+ sign Up on https://hub.docker.com/
+ create a repository --> https://hub.docker.com/repository/docker/elenaminyaeva/realworld
+ docker login
```
docker login -u elenaminyaeva -p Zxcvbn123
```
+ copy image ID
```
root@node1:/etc/kubernetes/node-express-realworld-example-app# docker images
REPOSITORY                                         TAG                 IMAGE ID            CREATED             SIZE
elenaminyaeva/realworld                            elena_images        b11beea1f590        4 days ago          212MB
realworld                                          latest              b11beea1f590        4 days ago          212MB
<none>                                             <none>              8747ab92ebdf        4 days ago          212MB
react-kubernetes                                   latest              40c02b7d2cd5        5 days ago          358MB
nginx                                              1.19                0901fa9da894        2 weeks ago         132MB
mongo                                              latest              6d11486a97a7        2 weeks ago         388MB
k8s.gcr.io/kube-proxy                              v1.18.5             a1daed4e2b60        4 weeks ago         117MB
k8s.gcr.io/kube-apiserver                          v1.18.5             08ca24f16874        4 weeks ago         173MB
k8s.gcr.io/kube-controller-manager                 v1.18.5             8d69eaf196dc        4 weeks ago         162MB
k8s.gcr.io/kube-scheduler                          v1.18.5             39d887c6621d        4 weeks ago         95.3MB
kubernetesui/dashboard-amd64                       v2.0.2              05126876084d        5 weeks ago         225MB
calico/node                                        v3.15.0             e0564c180e7c        5 weeks ago         262MB
calico/cni                                         v3.15.0             589bf4b0231f        5 weeks ago         217MB
calico/kube-controllers                            v3.15.0             5c0685e144e2        5 weeks ago         53.1MB
k8s.gcr.io/cluster-proportional-autoscaler-amd64   1.8.1               17ffd2ee7ad8        6 weeks ago         40.7MB
kubernetesui/metrics-scraper                       v1.0.5              2cd72547f23f        7 weeks ago         36.7MB
k8s.gcr.io/k8s-dns-node-cache                      1.15.13             3f7a09f7cade        2 months ago        107MB
paulbouwer/hello-kubernetes                        1.8                 444fd83eb497        3 months ago        130MB
k8s.gcr.io/pause                                   3.2                 80d28bedfe5d        5 months ago        683kB
coredns/coredns                                    1.6.7               67da37a9a360        6 months ago        43.8MB
node                                               13.6.0-alpine       2d8f48ba52b1        6 months ago        112MB
quay.io/coreos/etcd                                v3.4.3              a0b920cf970d        9 months ago        83.6MB
```

+ add tag
```
docker tag b11beea1f590 elenaminyaeva/realworld:elena_images
```
+ push image to docker hub
```
docker push elenaminyaeva/realworld:elena_images
```

+ add secret docker-registry
```
root@node1:/etc/kubernetes/node-express-realworld-example-app# export DOCKER_REGISTRY_SERVER=https://index.docker.io/v1/
root@node1:/etc/kubernetes/node-express-realworld-example-app# export DOCKER_USER=elenaminyaeva
root@node1:/etc/kubernetes/node-express-realworld-example-app# export DOCKER_EMAIL=alu0101251981@ull.edu.es
root@node1:/etc/kubernetes/node-express-realworld-example-app# export DOCKER_PASSWORD=Zxcvbn123
root@node1:/etc/kubernetes/node-express-realworld-example-app# kubectl create secret docker-registry elenaminyaeva \
> --docker-server=$DOCKER_REGISTRY_SERVER \
> --docker-username=$DOCKER_USER \
> --docker-password=$DOCKER_PASSWORD \
> --docker-email=$DOCKER_EMAIL
secret/elenaminyaeva created
```

+ change manifest file (.yaml)
```
spec:
      containers:
      - name: realworld
        image: index.docker.io/elenaminyaeva/realworld:elena_images
        ports:
        - containerPort: 8000
      imagePullSecrets:
      - name: elenaminyaeva
```

+ reconfigure pod
```
kubectl apply -f realwold.yaml
```

Repeat steps for **react-kubernetes** image

![Docker](/images/14.png)

### Problem SOLVED 

**kubectl logs -p realworld-d8b559548-srnfd**

> conduit-node@1.0.0 start /usr/src/app
> node ./app.js

Listening on port 3000

/usr/src/app/node_modules/mongodb/lib/server.js:242
        process.nextTick(function() { throw err; })
                                      ^
Error: connect ECONNREFUSED 127.0.0.1:27017
    at TCPConnectWrap.afterConnect [as oncomplete] (net.js:1137:16)
Emitted 'error' event on NativeConnection instance at:
    at NativeConnection.Connection.error (/usr/src/app/node_modules/mongoose/lib/connection.js:443:8)
    at /usr/src/app/node_modules/mongoose/lib/connection.js:472:15
    at /usr/src/app/node_modules/mongoose/lib/drivers/node-mongodb-native/connection.js:59:21
    at /usr/src/app/node_modules/mongodb/lib/db.js:232:14
    at Server.<anonymous> (/usr/src/app/node_modules/mongodb/lib/server.js:240:9)
    at Object.onceWrapper (events.js:428:26)
    at Server.emit (events.js:321:20)
    at Pool.<anonymous> (/usr/src/app/node_modules/mongodb-core/lib/topologies/server.js:308:68)
    at Pool.emit (events.js:321:20)
    at Connection.<anonymous> (/usr/src/app/node_modules/mongodb-core/lib/connection/pool.js:115:12)
    at Object.onceWrapper (events.js:428:26)
    at Connection.emit (events.js:321:20)
    at Socket.<anonymous> (/usr/src/app/node_modules/mongodb-core/lib/connection/connection.js:144:49)
    at Object.onceWrapper (events.js:428:26)
    at Socket.emit (events.js:321:20)
    at emitErrorNT (internal/streams/destroy.js:84:8) {
  name: 'MongoError',
  message: 'connect ECONNREFUSED 127.0.0.1:27017'
}
npm ERR! code ELIFECYCLE
npm ERR! errno 1
npm ERR! conduit-node@1.0.0 start: `node ./app.js`
npm ERR! Exit status 1
npm ERR! 
npm ERR! Failed at the conduit-node@1.0.0 start script.
npm ERR! This is probably not a problem with npm. There is likely additional logging output above.

npm ERR! A complete log of this run can be found in:
npm ERR!     /home/node/.npm/_logs/2020-07-23T12_18_18_608Z-debug.log

---
**SOLUTION**

Change app.js file --> change mongo host from *localhost* to *mongo*
```
if(isProduction){
  mongoose.connect(process.env.MONGODB_URI);
} else {
  mongoose.connect('mongodb://mongo/conduit');
  mongoose.set('debug', true);
}
```

##

### Set up MetalLB Load Balancing

```
kubectl apply -f https://raw.githubusercontent.com/metallb/metallb/v0.9.3/manifests/namespace.yaml
```
```
root@node1:/home/usuario# kubectl get ns
NAME              STATUS   AGE
default           Active   5d11h
kube-node-lease   Active   5d11h
kube-public       Active   5d11h
kube-system       Active   5d11h
metallb-system    Active   64s
mongodb           Active   5d7h
```

```
kubectl apply -f https://raw.githubusercontent.com/metallb/metallb/v0.9.3/manifests/metallb.yaml
```

```
root@node1:/etc/kubernetes# kubectl get pods -n metallb-system
NAME                          READY   STATUS    RESTARTS   AGE
controller-57f648cb96-bzhcx   1/1     Running   0          14h
speaker-cpzg4                 1/1     Running   0          14h
speaker-l7z7w                 1/1     Running   0          14h
speaker-lmfrr                 1/1     Running   0          14h
speaker-tb97p                 1/1     Running   0          14h
```

Create Configmap
```
vi configmap.yaml

> apiVersion: v1
> kind: ConfigMap
> metadata:
>   namespace: metallb-system
>   name: config
> data:
>   config: |
>     address-pools:
>     - name: default
>       protocol: layer2
>       addresses:
>       - 192.168.20.240-192.168.20.250
```

```
kubectl create -f configmap.yaml
```
```
root@node1:/etc/kubernetes# kubectl describe configmap config -nmetallb-system
Name:         config
Namespace:    metallb-system
Labels:       <none>
Annotations:  <none>

Data
====
config:
----
address-pools:
- name: default
  protocol: layer2
  addresses:
  - 192.168.20.240-192.168.20.250
```

Result --> External-IP
```
root@node1:/etc/kubernetes# kubectl get service realworld
NAME        TYPE           CLUSTER-IP      EXTERNAL-IP      PORT(S)        AGE
realworld   LoadBalancer   10.233.24.220   192.168.20.241   80:31645/TCP   4d8h

root@node1:/etc/kubernetes# kubectl get service react-kubernetes
NAME               TYPE           CLUSTER-IP      EXTERNAL-IP      PORT(S)        AGE
react-kubernetes   LoadBalancer   10.233.17.239   192.168.20.242   80:32518/TCP   5d9h
```
## NodePort Service

### react-new-try.yaml 

```
apiVersion: v1
kind: Service
metadata:
  name: react-new-try
  labels:
    app: react-new-try
spec:
  type: NodePort
  ports:
  - port: 80
    targetPort: 4100
  selector:
    app: react-new-try
```

```
# kubectl describe svc react-new-try
Name:                     react-new-try
Namespace:                default
Labels:                   app=react-new-try
Annotations:              Selector:  app=react-new-try
Type:                     NodePort
IP:                       10.233.33.202
Port:                     <unset>  80/TCP
TargetPort:               4100/TCP
NodePort:                 <unset>  31518/TCP
Endpoints:                10.233.90.36:4100,10.233.92.33:4100,10.233.96.20:4100
Session Affinity:         None
External Traffic Policy:  Cluster
```

*Check Results*
+ Accessing using Pod IP

```
react-new-try-6ffdf667b4-wv7nk               1/1     Running   0          57s     10.233.96.20    node2    <none>           <none>
react-new-try-6ffdf667b4-xg857               1/1     Running   0          40s     10.233.92.33    node3    <none>           <none>
react-new-try-6ffdf667b4-z7cgs               1/1     Running   0          49s     10.233.90.36    node1    <none>           <none>
```
```
# curl http://10.233.90.36:4100
# curl http://10.233.92.33:4100
# curl http://10.233.96.20:4100
```
PORT=targetPort=containerPort

+ Accessing using Node IP

```
NAME    STATUS   ROLES    AGE   VERSION   INTERNAL-IP     EXTERNAL-IP   OS-IMAGE             KERNEL-VERSION       CONTAINER-RUNTIME
node1   Ready    master   9d    v1.18.5   192.168.20.36   <none>        Ubuntu 18.04.2 LTS   4.15.0-112-generic   docker://19.3.11
node2   Ready    master   9d    v1.18.5   192.168.20.37   <none>        Ubuntu 18.04.2 LTS   4.15.0-50-generic    docker://19.3.11
node3   Ready    <none>   9d    v1.18.5   192.168.20.34   <none>        Ubuntu 18.04.2 LTS   4.15.0-50-generic    docker://19.3.11
node4   Ready    <none>   9d    v1.18.5   192.168.20.35   <none>        Ubuntu 18.04.2 LTS   4.15.0-112-generic   docker://19.3.11
```
```
#curl http://192.168.20.36:31518
#curl http://192.168.20.37:31518
#curl http://192.168.20.34:31518
```

PORT=nodePort

+ Accessing using Service IP

```
NAME                        TYPE           CLUSTER-IP      EXTERNAL-IP      PORT(S)                      AGE
react-new-try               NodePort       10.233.33.202   <none>           80:31518/TCP                 2m9s
```
```
# curl http://10.233.33.202:80
```

PORT=port

### realworld.yaml

+ Accessing using Pod IP
```
# curl http://10.233.105.33:3000/api/articles
```
+ Accessing using Service IP

```
curl http://10.233.24.220:80/api/articles
```


## Install Helm 

```
curl https://helm.baltorepo.com/organization/signing.asc | sudo apt-key add -
sudo apt-get install apt-transport-https --yes
echo "deb https://baltocdn.com/helm/stable/debian/ all main" | sudo tee /etc/apt/sources.list.d/helm-stable-debian.list
sudo apt-get update
sudo apt-get install helm
```
```
root@node1:/home/usuario# helm version --short
v3.2.4+g0ad800e
```

## Install ingress

https://docs.nginx.com/nginx-ingress-controller/installation/installation-with-helm/

```
git clone https://github.com/nginxinc/kubernetes-ingress/
```
```
cd kubernetes-ingress/deployments/helm-chart
```
```
helm repo add nginx-stable https://helm.nginx.com/stable
helm repo update
```
```
/helm-chart# helm install my-release nginx-stable/nginx-ingress
NAME: my-release
LAST DEPLOYED: Tue Jul 28 11:35:59 2020
NAMESPACE: default
STATUS: deployed
REVISION: 1
TEST SUITE: None
NOTES:
The NGINX Ingress Controller has been installed.
```

```
root@node1:/etc/kubernetes/react-redux-realworld-example-app# kubectl get pods -o wide
NAME                                         READY   STATUS        RESTARTS   AGE     IP              NODE     NOMINATED NODE   READINESS GATES
...
my-release-nginx-ingress-6b6d84b496-gwdc8    1/1     Running       0          7h48m   10.233.96.12    node2    <none>           <none>
```
```
root@node1:/etc/kubernetes# kubectl get svc
NAME                        TYPE           CLUSTER-IP      EXTERNAL-IP      PORT(S)                      AGE
...
my-release-nginx-ingress    LoadBalancer   10.233.54.123   192.168.20.244   80:31794/TCP,443:31500/TCP   7h54m
```

## Configure HAProxy

VM IP 192.168.20.11

```
# yum install -y haproxy
```
```
# cat /etc/haproxy/haproxy.cfg 
#---------------------------------------------------------------------
# Example configuration for a possible web application.  See the
# full configuration options online.
#
#   http://haproxy.1wt.eu/download/1.4/doc/configuration.txt
#
#---------------------------------------------------------------------

#---------------------------------------------------------------------
# Global settings
#---------------------------------------------------------------------
global
    # to have these messages end up in /var/log/haproxy.log you will
    # need to:
    #
    # 1) configure syslog to accept network log events.  This is done
    #    by adding the '-r' option to the SYSLOGD_OPTIONS in
    #    /etc/sysconfig/syslog
    #
    # 2) configure local2 events to go to the /var/log/haproxy.log
    #   file. A line like the following can be added to
    #   /etc/sysconfig/syslog
    #
    #    local2.*                       /var/log/haproxy.log
    #
    log         127.0.0.1 local2

    chroot      /var/lib/haproxy
    pidfile     /var/run/haproxy.pid
    maxconn     4000
    user        haproxy
    group       haproxy
    daemon

    # turn on stats unix socket
    stats socket /var/lib/haproxy/stats

#---------------------------------------------------------------------
# common defaults that all the 'listen' and 'backend' sections will
# use if not designated in their block
#---------------------------------------------------------------------
defaults
    mode                    http
    log                     global
    option                  httplog
    option                  dontlognull
    option http-server-close
    option forwardfor       except 127.0.0.0/8
    option                  redispatch
    retries                 3
    timeout http-request    10s
    timeout queue           1m
    timeout connect         10s
    timeout client          1m
    timeout server          1m
    timeout http-keep-alive 10s
    timeout check           10s
    maxconn                 3000

frontend http_front
  bind *:80
  stats uri /haproxy?stats
  default_backend http_back

backend http_back
  balance roundrobin
  server kube 192.168.20.34:80
  server kube 192.168.20.35:80
  ```

## Configure Ingress for example app

https://github.com/justmeandopensource/kubernetes/tree/master/yamls/ingress-demo

```
# cat ingress-resource-new.yaml 
apiVersion: networking.k8s.io/v1beta1
kind: Ingress
metadata:
  name: ingress-resource-1
spec:
  rules:
  - host: nginx.example.com
    http:
      paths:
      - backend:
          serviceName: nginx-deploy-main
          servicePort: 80
````
```
# cat nginx-deploy-main.yaml 
apiVersion: apps/v1
kind: Deployment
metadata:
  labels:
    run: nginx
  name: nginx-deploy-main
spec:
  replicas: 1
  selector:
    matchLabels:
      run: nginx-main
  template:
    metadata:
      labels:
        run: nginx-main
    spec:
      containers:
      - image: nginx
        name: nginx
```

Update /etc/hosts

```
192.168.20.11 nginx.example.com
```
Result 

![Docker](/images/15.png)



## Configure Ingress for realworld

1. Create an internal service --> react-kubernetes-internal.yaml 

```
root@node1:/etc/kubernetes/react-redux-realworld-example-app# cat react-kubernetes-internal.yaml
apiVersion: v1
kind: Service
metadata:
  name: react-kubernetes-internal
spec:
  ports:
  - port: 80
    targetPort: 4100
  selector:
    app: react-kubernetes-internal
---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: react-kubernetes-internal
spec:
  replicas: 4
  selector:
    matchLabels:
      app: react-kubernetes-internal
  template:
    metadata:
      labels:
        app: react-kubernetes-internal
    spec:
      containers:
      - name: react-kubernetes-internal
        image: index.docker.io/elenaminyaeva/react:elena_images
        ports:
        - containerPort: 4100
      imagePullSecrets:
      - name: elenaminyaeva
      hostNetwork: true
      dnsPolicy: Default  
```

```
root@node1:/etc/kubernetes/react-redux-realworld-example-app# kubectl get svc
NAME                        TYPE           CLUSTER-IP      EXTERNAL-IP      PORT(S)                      AGE
..
react-kubernetes-internal   ClusterIP      10.233.9.23     <none>           80/TCP                     7h31m
```

*no external-ip as internal service*

2. Create ingress --> ingress-react.yaml

```
apiVersion: networking.k8s.io/v1beta1
kind: Ingress
metadata:
  annotations:
    nginx.ingress.kubernetes.io/use-regex: "true"
  name: ingress-resource-1
spec:
  rules:
  - host: nginx.example.com
    http:
      paths:
      - backend:
          serviceName: nginx-deploy-main
          servicePort: 80
  - host: react.example.com
    http:
      paths:
      - backend:
          serviceName: react-kubernetes-internal
          servicePort: 80
```

```
root@node1:/etc/kubernetes# kubectl get ing
NAME                 CLASS    HOSTS                                 ADDRESS          PORTS   AGE
ingress-resource-1   <none>   nginx.example.com,react.example.com   192.168.20.244   80      35m
```


3. Update /etc/hosts

```
192.168.20.11 nginx.example.com
192.168.20.11 react.example.com
192.168.20.11 realworld.example.com
```
![Docker](/images/16.png)
![Docker](/images/17.png)
### Problem SOLVED

502 Bad Gateway error for react.example.com

```
# kubectl get all -n nginx-ingress
NAME                      READY   STATUS    RESTARTS   AGE
pod/nginx-ingress-989mv   1/1     Running   2          3h45m
pod/nginx-ingress-c7252   1/1     Running   0          3h45m
pod/nginx-ingress-wz99x   1/1     Running   0          3h45m
pod/nginx-ingress-xnhl7   1/1     Running   0          3h45m
```
```
root@node1:/home/usuario# kubectl logs nginx-ingress-989mv -n nginx-ingress
```

```
2020/07/29 13:33:14 [error] 215#215: *112 connect() failed (111: Connection refused) while connecting to upstream, client: 192.168.20.11, server: react.nginx.example.com, request: "GET / HTTP/1.1", upstream: "http://10.233.90.31:80/", host: "react.nginx.example.com"
```

*Solution*

Change targetPort=containerPort=appPort (according to package.json)

"scripts": {
    "start": "cross-env PORT=4100 react-scripts start",
    "build": "react-scripts build",
    "test": "cross-env PORT=4100 react-scripts test --env=jsdom",
    "eject": "react-scripts eject"
  },


# K8s in GKE

1. Install Google Cloud SDK
https://cloud.google.com/sdk/docs/quickstart-macos

2. Iniciate SDK

```
gcloud init
```
3. Set up a project

```
gcloud config set project elena-test-project-a966b
```

4. Install *kubectl*

```
brew install kubernetes-cli
```
```
lenaminyaeva@MacBook-Pro-de-Lena ~ % kubectl version --client
Client Version: version.Info{Major:"1", Minor:"18", GitVersion:"v1.18.6", GitCommit:"dff82dc0de47299ab66c83c626e08b245ab19037", GitTreeState:"clean", BuildDate:"2020-07-16T00:04:31Z", GoVersion:"go1.14.4", Compiler:"gc", Platform:"darwin/amd64"}
```

5. Create a cluster

```
gcloud services enable container.googleapis.com
```
```
gcloud container clusters create elena-k8s --zone europe-west3 --num-nodes=2
```
```
kubeconfig entry generated for elena-k8s.
NAME       LOCATION      MASTER_VERSION  MASTER_IP       MACHINE_TYPE   NODE_VERSION   NUM_NODES  STATUS
elena-k8s  europe-west3  1.15.12-gke.2   35.246.178.151  n1-standard-1  1.15.12-gke.2  6          RUNNING
```
**Cluster**
![GCloud](/images/18.png)

**VMs**
![GCloud](/images/19.png)

6. Cluster configuarion

```
lenaminyaeva@MacBook-Pro-de-Lena ~ % gcloud container clusters get-credentials elena-k8s --zone europe-west3 
Fetching cluster endpoint and auth data.
kubeconfig entry generated for elena-k8s.
```
```
lenaminyaeva@MacBook-Pro-de-Lena ~ % kubectl cluster-info
Kubernetes master is running at https://35.246.178.151
GLBCDefaultBackend is running at https://35.246.178.151/api/v1/namespaces/kube-system/services/default-http-backend:http/proxy
Heapster is running at https://35.246.178.151/api/v1/namespaces/kube-system/services/heapster/proxy
KubeDNS is running at https://35.246.178.151/api/v1/namespaces/kube-system/services/kube-dns:dns/proxy
Metrics-server is running at https://35.246.178.151/api/v1/namespaces/kube-system/services/https:metrics-server:/proxy
```

```
lenaminyaeva@MacBook-Pro-de-Lena ~ % kubectl get nodes
NAME                                       STATUS   ROLES    AGE   VERSION
gke-elena-k8s-default-pool-50406d1c-8zw3   Ready    <none>   11m   v1.15.12-gke.2
gke-elena-k8s-default-pool-50406d1c-fx53   Ready    <none>   11m   v1.15.12-gke.2
gke-elena-k8s-default-pool-ab5003d1-cwht   Ready    <none>   11m   v1.15.12-gke.2
gke-elena-k8s-default-pool-ab5003d1-d3bh   Ready    <none>   11m   v1.15.12-gke.2
gke-elena-k8s-default-pool-dea0cd3b-c43h   Ready    <none>   11m   v1.15.12-gke.2
gke-elena-k8s-default-pool-dea0cd3b-tds0   Ready    <none>   11m   v1.15.12-gke.2
```

Note: *num-nodes es la cantidad de nodos del grupo en un clúster zonal. Si usas clústeres multizonales o regionales, nun-nodes es la cantidad de nodos para cada zona en la que se encuentran los grupos de nodos.*

Assign roles
```
kubectl label node *node name* node-role.kubernetes.io/*role*=*role*
```
```
lenaminyaeva@MacBook-Pro-de-Lena ~ % kubectl get nodes -o wide                                                                        
NAME                                       STATUS   ROLES    AGE   VERSION          INTERNAL-IP   EXTERNAL-IP      OS-IMAGE                             KERNEL-VERSION   CONTAINER-RUNTIME
gke-elena-k8s-default-pool-50406d1c-8zw3   Ready    worker   91m   v1.15.12-gke.2   10.156.0.3    34.107.67.175    Container-Optimized OS from Google   4.19.112+        docker://19.3.1
gke-elena-k8s-default-pool-50406d1c-fx53   Ready    master   91m   v1.15.12-gke.2   10.156.0.2    34.107.85.39     Container-Optimized OS from Google   4.19.112+        docker://19.3.1
gke-elena-k8s-default-pool-ab5003d1-cwht   Ready    master   91m   v1.15.12-gke.2   10.156.0.4    34.89.209.85     Container-Optimized OS from Google   4.19.112+        docker://19.3.1
gke-elena-k8s-default-pool-ab5003d1-d3bh   Ready    worker   91m   v1.15.12-gke.2   10.156.0.7    34.107.32.230    Container-Optimized OS from Google   4.19.112+        docker://19.3.1
gke-elena-k8s-default-pool-dea0cd3b-c43h   Ready    worker   91m   v1.15.12-gke.2   10.156.0.5    35.242.192.202   Container-Optimized OS from Google   4.19.112+        docker://19.3.1
gke-elena-k8s-default-pool-dea0cd3b-tds0   Ready    worker   91m   v1.15.12-gke.2   10.156.0.6    35.234.102.146   Container-Optimized OS from Google   4.19.112+        docker://19.3.1
```

## Try out cypress test app locally

![GCloud](/images/20.png)

## Deploy cypress test app in cluster

+ build docker image
+ push image to docker hub
![GCloud](/images/21.png)

+ deploy image

![GCloud](/images/22.png)

![GCloud](/images/23.png)

![GCloud](/images/24.png)

### Problem SOLVED: Pods crashing

![GCloud](/images/24.png)

```
lenaminyaeva@MacBook-Pro-de-Lena game % kubectl get pods -o wide                  
NAME                            READY   STATUS             RESTARTS   AGE     IP          NODE                                       NOMINATED NODE   READINESS GATES
conduit-test-74cdfc49bb-8njf5   0/1     CrashLoopBackOff   725        2d19h   10.16.4.4   gke-elena-k8s-default-pool-dea0cd3b-c43h   <none>           <none>
conduit-test-74cdfc49bb-bl4n4   0/1     CrashLoopBackOff   731        2d19h   10.16.6.4   gke-elena-k8s-default-pool-dea0cd3b-tds0   <none>           <none>
conduit-test-74cdfc49bb-dvplt   0/1     CrashLoopBackOff   728        2d19h   10.16.2.4   gke-elena-k8s-default-pool-50406d1c-fx53   <none>           <none>
conduit-test-74cdfc49bb-lxgpq   0/1     CrashLoopBackOff   721        2d19h   10.16.1.5   gke-elena-k8s-default-pool-ab5003d1-cwht   <none>           <none>
conduit-test-74cdfc49bb-s2tt8   1/1     Running            726        2d19h   10.16.3.4   gke-elena-k8s-default-pool-50406d1c-8zw3   <none>           <none>
````

```
lenaminyaeva@MacBook-Pro-de-Lena game % kubectl describe pod conduit-test-74cdfc49bb-lxgpq
Name:           conduit-test-74cdfc49bb-lxgpq
Namespace:      default
Priority:       0
Node:           gke-elena-k8s-default-pool-ab5003d1-cwht/10.156.0.4
Start Time:     Sun, 09 Aug 2020 14:12:11 +0100
Labels:         app=conduit-test
                pod-template-hash=74cdfc49bb
Annotations:    kubernetes.io/limit-ranger: LimitRanger plugin set: cpu request for container conduit-test-1
Status:         Running
IP:             10.16.1.5
IPs:            <none>
Controlled By:  ReplicaSet/conduit-test-74cdfc49bb
Containers:
  conduit-test-1:
    Container ID:   docker://056568616b94374ab6bcf899508ddc2744ced7e2e6a49b19ade06543351f8210
    Image:          index.docker.io/elenaminyaeva/conduit-test:latest
    Image ID:       docker-pullable://elenaminyaeva/conduit-test@sha256:8a61d7fdc491743d9cf38a04ba6566dee5fe75d879a243542cb0890ce5dbd08c
    Port:           <none>
    Host Port:      <none>
    State:          Waiting
      Reason:       CrashLoopBackOff
    Last State:     Terminated
      Reason:       Error
      Exit Code:    1
      Started:      Wed, 12 Aug 2020 09:21:29 +0100
      Finished:     Wed, 12 Aug 2020 09:21:58 +0100
    Ready:          False
    Restart Count:  722
    Requests:
      cpu:        100m
    Environment:  <none>
    Mounts:
      /var/run/secrets/kubernetes.io/serviceaccount from default-token-j8g54 (ro)
Conditions:
  Type              Status
  Initialized       True 
  Ready             False 
  ContainersReady   False 
  PodScheduled      True 
Volumes:
  default-token-j8g54:
    Type:        Secret (a volume populated by a Secret)
    SecretName:  default-token-j8g54
    Optional:    false
QoS Class:       Burstable
Node-Selectors:  <none>
Tolerations:     node.kubernetes.io/not-ready:NoExecute for 300s
                 node.kubernetes.io/unreachable:NoExecute for 300s
Events:
  Type     Reason   Age                       From                                               Message
  ----     ------   ----                      ----                                               -------
  Normal   Pulling  36m (x717 over 2d19h)     kubelet, gke-elena-k8s-default-pool-ab5003d1-cwht  Pulling image "index.docker.io/elenaminyaeva/conduit-test:latest"
  Warning  BackOff  113s (x16850 over 2d19h)  kubelet, gke-elena-k8s-default-pool-ab5003d1-cwht  Back-off restarting failed container
```
```
lenaminyaeva@MacBook-Pro-de-Lena game % kubectl describe pod conduit-test-74cdfc49bb-lxgpq
...
[start:server:coverage] Error: /usr/src/app/server/node_modules/sodium-native/build/Release/sodium.node: invalid ELF header
...
```

**Solution 1**

*Build image on Linux machine*

```
conduit-new-d96d46897-bp4k5     0/1     CrashLoopBackOff   3          82s     10.16.2.5   gke-elena-k8s-default-pool-50406d1c-fx53   <none>           <none>
conduit-new-d96d46897-klkz2     0/1     CrashLoopBackOff   3          82s     10.16.4.6   gke-elena-k8s-default-pool-dea0cd3b-c43h   <none>           <none>
conduit-new-d96d46897-sthn8     0/1     CrashLoopBackOff   3          82s     10.16.1.7   gke-elena-k8s-default-pool-ab5003d1-cwht   <none>           <none>
```

```
lenaminyaeva@MacBook-Pro-de-Lena game % kubectl logs conduit-new-d96d46897-bp4k5

> realworld@1.0.0 start /usr/src/app
> concurrently npm:start:client npm:start:server

sh: 1: concurrently: not found
npm ERR! code ELIFECYCLE
npm ERR! syscall spawn
npm ERR! file sh
npm ERR! errno ENOENT
npm ERR! realworld@1.0.0 start: `concurrently npm:start:client npm:start:server`
npm ERR! spawn ENOENT
npm ERR! 
npm ERR! Failed at the realworld@1.0.0 start script.
npm ERR! This is probably not a problem with npm. There is likely additional logging output above.

npm ERR! A complete log of this run can be found in:
npm ERR!     /root/.npm/_logs/2020-08-12T12_40_21_709Z-debug.log
```

*Extend Dockerfile*

```
FROM node:12
  
# Create app directory
WORKDIR /usr/src/app

# Install app dependencies
# A wildcard is used to ensure both package.json AND package-lock.json are copied
# where available (npm@5+)
COPY package*.json ./
COPY client/package*.json ./
COPY server/package*.json ./

RUN npm install -g npm@latest
RUN npm install
RUN npm install -g parcel-bundler
# If you are building your code for production
# RUN npm ci --only=production

# Bundle app source
COPY . .

EXPOSE 4100
CMD [ "npm", "start" ]
```
![GCloud](/images/26.png)

```
conduit-supernew-7d559bd984-2g2l6   1/1     Running            0          5m47s   10.16.3.7   gke-elena-k8s-default-pool-50406d1c-8zw3   <none>           <none>
conduit-supernew-7d559bd984-nxk7x   1/1     Running            0          3m46s   10.16.2.8   gke-elena-k8s-default-pool-50406d1c-fx53   <none>           <none>
conduit-supernew-7d559bd984-rfwfk   1/1     Running            0          3m46s   10.16.1.8   gke-elena-k8s-default-pool-ab5003d1-cwht   <none>           <none>
conduit-supernew-7d559bd984-vbsgz   1/1     Running            0          5m47s   10.16.6.6   gke-elena-k8s-default-pool-dea0cd3b-tds0   <none>           <none>
conduit-supernew-7d559bd984-xb6w6   1/1     Running            0          5m47s   10.16.4.7   gke-elena-k8s-default-pool-dea0cd3b-c43h   <none>           <none>
```

## Check functionality

**Expose service**

```
lenaminyaeva@MacBook-Pro-de-Lena server % kubectl get svc
NAME                       TYPE           CLUSTER-IP      EXTERNAL-IP     PORT(S)        AGE
conduit-supernew-service   LoadBalancer   10.19.246.8     35.198.86.120   80:30431/TCP   86s
conduit-test-service       LoadBalancer   10.19.245.148   34.107.48.37    80:30936/TCP   3d20h
kubernetes                 ClusterIP      10.19.240.1     <none>          443/TCP        5d22h
ngnix-deploy-87lfq         LoadBalancer   10.19.240.213   34.89.145.47    80:32265/TCP   5d19h
```

