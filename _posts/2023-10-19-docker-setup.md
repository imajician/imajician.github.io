---
layout: post
read_time: true
show_date: true
title: "Docker Setup Guide"
date: 2023-10-19
img: posts/20231019/docker_banner.png
tags: [copyright, creativity, virtual machine]
category: virtual machine
author: Imajician
description: "Quick guide to set up docker"
---

## Preparation
--------

[Official Installation Page](https://docs.docker.com/get-docker/)

[Portainer Installation](https://docs.portainer.io/contribute/build/mac)

<br>

## Installation 
-----------
- Standard installation
```bash
$ sudo apt-get install ca-certificates curl gnupg lsb-release -y
$ curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /usr/share/keyrings/docker-archive-keyring.gpg
$ echo "deb [arch=$(dpkg --print-architecture) signed-by=/usr/share/keyrings/docker-archive-keyring.gpg] https://download.docker.com/linux/ubuntu $(lsb_release -cs) stable" | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
$ sudo apt-get update
$ sudo apt-get install docker-ce docker-ce-cli containerd.io
$ sudo usermod -aG docker $USER
```

- MacOS Installation with Homebrew:
```bash
$ brew install --cask docker
```
<br>

## Network
-----------------
### Default Bridge

1. check available networks:
```bash
$ docker network ls
NETWORK ID     NAME      DRIVER    SCOPE
5077a7b25ae6   bridge    bridge    local
7e25f334b07f   host      host      local
475e50be0fe0   none      null      local
```

2. start two containers:
```bash
$ docker run -dit --name busybox1 busybox /bin/sh
$ docker run -dit -p 80:80 --name busybox2 busybox /bin/sh
$ docker ps
CONTAINER ID   IMAGE     COMMAND     CREATED          STATUS          PORTS     NAMES
9e6464e82c4c   busybox   "/bin/sh"   5 seconds ago    Up 5 seconds              busybox2
7fea14032748   busybox   "/bin/sh"   26 seconds ago   Up 26 seconds             busybox1 
```

4. Verify the containers are attached to the bridge network
```bash
$ docker network inspect bridge
```

### User-Defined Bridge
1. Create and remove network
```bash
$ docker network create mynetwork1
$ docker network ls
$ docker network remove mynetwork1
```

2. Run container:
```bash
$ docker run -itd --rm --network mynetwork1 --name busybox3 busybox 
$ docker run -itd --rm --network mynetwork1 --name busybox4 busybox 
$ docker ps
$ docker network inspect mynetwork1
```

### Host
```bash
$ docker run -itd --rm --network host --name busybox5 nginx
// ports exposed, no network isolation
```

### MACVLAN
1. Create macvlan
```bash
$ docker network create -d macvlan --subnet 192.168.0.0/24 --gateway 192.168.0.1 -o parent=enp0s3 --name mynetwork2 
```

2. Enable promiscous mode:
```bash
$ sudo ip link set enp0s3 promisc on
// also enable promiscous mode in virtualbox
```

3. Run container on this network
```bash
$ docker run -itd --rm --network mynetwork2 --ip 192.168.0.123 --name busybox6 busybox 
$ docker run -itd --rm --network mynetwork2 --ip 192.168.0.124 --name busybox6 nginx 
$ docker  exec -it busybox6 sh
/# ping 192.168.0.1  
```

4. Remove the network
```bash
$ docker network remove mynetwork2
```
5. Alternative:
```bash
 $ docker network create -d macvlan --subnet 192.168.20.0/24 --gateway 192.168.20.1 -o parent=enp0s3.20 --name mynetwork2    
```

### IPVLAN
1. Share Mac Address with host, with different IP addresses
```bash
$ docker network create -d ipvlan --subnet 192.168.0.0/24 --gateway 192.168.0.1 -o parent=enp0s3 --name mynetwork3 
```

2. Use the host as a router
```bash
docker network create -d ipvlan --subnet 192.168.94.0/24 -o parent=enp0s3 -o ipvlan_mode=l3 --subnet 192.168.95.0/24 --name mynetwork4 
```

3. Assign IP to container
```bash
$ docker run -itd --rm --network mynetwork4 --ip 192.168.94.7 --busybox7 busybox
$ docker run -itd --rm --network mynetwork4 --ip 192.168.94.8 --busybox8 busybox
```

4. Inspect network
```bash
docker network inspect mynetwork4
```

5. Establish static route in router 

### Overlay Network 
- Standalone overlay network
```bash
$ docker network create -d overlay --name my-overlay-network
```

- Attachable overlay network
```bash
$ docker network create -d overlay --attachable --name my-attachable-overlay
```

### None network
```bash
$ docker run -itd --rm --network none --name busybox9 busybox
```

<br>

## Portainer
-----------------

### Build and Deploy Container

1. Create persistent volum:
```bash
$ docker volume create portainer_data
```

2. Deploy container:
```bash
$ docker run -d -p 8000:8000 -p 9000:9000 --name=portainer --restart=always -v /var/run/docker.sock:/var/run/docker.sock -v portainer_data:/data portainer/portainer
```

3. URL: 
>http://localhost:9000
   
<br>

### Update
1. Remove old version:
```bash
$ docker ps -a | portainer
$ docker stop 0eab
$ docker rm 0eab
```

2. Install new version:
```bash
$ docker pull portainer/portainer-ce:latest
$ docker run -d -p 8000:8000 -p 9443:9443 --name=portainer --restart=always -v /var/run/docker.sock:/var/run/docker.sock -v portainer_data:/data portainer/portainer-ce
```

3. URL: 
>http://localhost:9443

