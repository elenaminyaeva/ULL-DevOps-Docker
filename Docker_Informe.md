# Infome Avances Docker Swarm

## Environment Set Up

192.168.20.20 --> Ansible Control Server (1)
192.168.20.27 --> Swarm Manager Server  (2)

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

5. Install Docket on (2) and check the installed version
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
IP --> 192.168.20.28 (3)

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
IP --> 192.168.20.29 (4)

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
Ceck current state 
```
ID                            HOSTNAME            STATUS              AVAILABILITY        MANAGER STATUS      ENGINE VERSION
21zfgkavepn2z349ac3m31o3j *   ubuntu              Ready               Active              Leader              19.03.12
mjjl40vjr6ye4uon8xi1e3aax     ubuntu              Ready               Active                                  19.03.12
uthfhenxsmmcbb2w49sa6rhz4     ubuntu              Ready               Active                                  19.03.12
```

## Deploy a service to the swarm

1. Run the following command on manager machine: 
```
# docker service create --replicas 1 --name helloworld alpine ping docker.com

ljo1n3ygvmvpzunufwrndu4ba
overall progress: 1 out of 1 tasks 
1/1: running   [==================================================>] 
verify: Service converged 
```
```
# docker service ls
ID                  NAME                MODE                REPLICAS            IMAGE               PORTS
ljo1n3ygvmvp        helloworld          replicated          1/1                 alpine:latest       
```


## DNS Problem

1. DNS Settings changed

![Docker](/images/7.png)

2. SSH server (2) fails with timeout error
 
3. Docker run fails on the *RUN NPM step* with the following error:

Nodejs App --> /var/tmp/Nodeja-application-with-docker

```
getaddrinfo EAI_AGAIN
```
![Docker](/images/5.png)

4. Docker restart fails with the following error:

```
Failed to start docker application container engine
```
![Docker](/images/6.png)



