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
```bash
$ docker network ls
```

### User-Defined Bridge
```bash
```

### MACVLAN
```bash
```

### IPVLAN
```bash
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

