---
layout: post
read_time: true
show_date: true
title: "Kasm Quick Guide"
date: 2022-08-15
img: posts/20220816/kasm.jpeg
tags: [copyright, creativity, virtual machine]
category: virtual machine
author: Imajician
description: "Quick guide to set up Kasm"
---

## Preparation
--------

[Official Installation Page](https://kasmweb.com/docs/latest/install/single_server_install.html)

### Swap Memory

```bash

$ sudo dd if=/dev/zero bs=1M count=4096 of=/mnt/4GiB.swap
$ sudo chmod 600 /mnt/4GiB.swap
$ sudo mkswap /mnt/4GiB.swap
$ sudo swapon /mnt/4GiB.swap
# make swap available on boot
$ echo '/mnt/4GiB.swap swap swap defaults 0 0' | sudo tee -a /etc/fstab
```

### Verify swap file exists
```bash
$ cat /proc/swaps
```
<br>

## Installation 
-----------
- Standard installation
```bash
$ cd /tmp
$ curl -O https://kasm-static-content.s3.amazonaws.com/kasm_release_1.11.0.18142e.tar.gz
$ tar -xf kasm_release*.tar.gz
$ sudo bash kasm_release/install.sh
```

- Save the credentials for default login
 
- If you would like to run the Web Application on a different port pass the -L flag when calling the installer. e.g **sudo bash kasm_release/install.sh -L 8443**

<br>

## Uninstallation
-------------

1. Stop All Kasm services.
```bash
$ sudo /opt/kasm/current/bin/stop
```

2. Remove any Kasm session containers.
```bash
$ sudo docker rm -f $(sudo docker container ls -qa --filter="label=kasm.kasmid")
```

3. Remove Kasm service containers.
```bash
$ export KASM_UID=$(id kasm -u)
$ export KASM_GID=$(id kasm -g)
$ sudo -E docker-compose -f /opt/kasm/current/docker/docker-compose.yaml rm
```

4. Remove the Kasm docker network.
```bash
$ sudo docker network rm kasm_default_network
```

5. Remove the Kasm database docker volume.
```bash
$ sudo docker volume rm kasm_db_1.11.0
```

6. Remove the Kasm docker images.
```bash
$ sudo docker rmi redis:5-alpine
$ sudo docker rmi postgres:9.5-alpine
$ sudo docker rmi kasmweb/nginx:latest
$ sudo docker rmi kasmweb/share:1.11.0
$ sudo docker rmi kasmweb/agent:1.11.0
$ sudo docker rmi kasmweb/manager:1.11.0
$ sudo docker rmi kasmweb/api:1.11.0
$ sudo docker rmi $(sudo docker images --filter "label=com.kasmweb.image=true" -q)
```

7. Remove the Kasm installation directory structure.
```bash
$ sudo rm -rf /opt/kasm/
```
<br>

## HTTPS Certificate
-----------------

### Steps: 

1. Become private CA by generating private Root Certificate.
2. Create new certificate for the usage of Kasm Workspaces.

### Acquire private Certificate Authority Root Certificate

1. Generate CA private key:
```bash
$ openssl genrsa -aes256 -out majician-ca.key 2048
```

2. Generate CA Root Certificate (10ys validity):
```bash
$ openssl req -x509 -new -nodes -key majician-ca.key -sha256 -days 3650 -out majician-ca.pem
```

3. ca-certificates expects PEM files with *.crt extension, so let's give it to him:
```bash
$ sudo cp majician-ca.pem /usr/local/share/ca-certificates/majician-ca.crt
```

4. Update certificates database and verify:
```bash
$ sudo update-ca-certificates
# sudo update-ca-certificates --fresh / to rebuild from scratch
$ awk -v cmd='openssl x509 -noout -subject' '/BEGIN/{close(cmd)};{print | cmd}' < /etc/ssl/certs/ca-certificates.crt | grep majician
```

### Acquire private certificate for Kasm Workspaces

1. Generate private key:
```bash
$ openssl genrsa -out kasm.rpi.key 2048
```

2. Create Certificate Signing Request (CSR):
```bash
$ openssl req -new -key kasm.rpi.key -out kasm.rpi.csr
```

3. Create ext file (kasm.rpi.ext) to supply during making a signing request.
```text
authorityKeyIdentifier=keyid,issuer
basicConstraints=CA:FALSE
keyUsage = digitalSignature, nonRepudiation, keyEncipherment, dataEncipherment
subjectAltName = @alt_names
[alt_names]
DNS.1 = kasm.rpi
```

4. Create signed certificate for the Kasm:
```bash
$ openssl x509 -req -in kasm.rpi.csr -CA majician-ca.pem -CAkey majician-ca.key -CAcreateserial -out kasm.rpi.crt -days 730 -sha256 -extfile kasm.rpi.ext
```


- Now you have the CRT certificate that can be used in application.
>You may notice that additional SRL file for Root CA is created. It is required by OpenSSL to track serial number of generated certificates - read more about it here.

### Upload certificates to Kasm and endpoints

1. SSH into Kasm server and replace certificate and the private key:
```bash
$ sudo /opt/kasm/bin/stop
$ sudo cp ~/.certs/kasm.rpi.crt /opt/kasm/current/certs/kasm_nginx.crt
$ sudo cp ~/.certs/kasm.rpi.key /opt/kasm/current/certs/kasm_nginx.key
$ sudo /opt/kasm/bin/start
```

2. Copy CA Root Certificate to the systems that will be using the Kasm Workspaces. It depends on the system, but on Windows - double-click the certificate and import it to Local Machine (or Current User) in Trusted Root Certification Authorities.

3. Change hosts entry (on endpoint) to point chosen Kasm address to the IP address.
```bash
192.168.1.100 kasm.rpi
```

4. Changing the Upstream Auth Address for the default zone to the local IP address.

