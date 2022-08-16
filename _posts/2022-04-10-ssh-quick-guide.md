---
layout: post
read_time: true
show_date: true
title: "SSH Quick Guide"
date: 2022-04-10
img: posts/20220410/ssh-big-825x510.png
tags: [copyright, creativity, neural networks, machine learning, artificial intelligence]
category: opinion
author: Imajician
description: "Quick guide to set up SSH"
---

## Generate new SSH Key

```bash
$ ssh-keygen -t rsa -b 4096 -C "name@email.com"
    ~/.ssh/id_rsa
    ~/.ssh/id-rsa.pub

    # (optional passphrase and name)
```

## Add new key to ssh-agent

```bash
$ eval $(ssh-agent -s)
($ eval `ssh-agent -s`) 
    Agent pid 12345

$ ssh-add ~/.ssh/id_rsa
```

#### For Windows, open PowerShell as Administrator

```bash
Add-WindowsCapability -Online -Name OpenSSH.Client*
```

OR

1. Open **Manage Optional Features**
1. **Add a Feature**
1. Search for **OpenSSH** and install
<br>

THEN

```bash
$ key-gen -t rsa -b 4096 -C "name@email.com"
    # C:\Users\user/.ssh/id_rsa
    # C:\Users\user/.ssh/id_rsa.pub
$ New-Item -Path 'C:/Users/user/.ssh/config' -ItemType File
```

## Copy SSH key to remote

#### Use ssh-copy-id (Mac Only)

```bash
brew install ssh-copy-id
ssh-copy-id demo@192.168.1.1
```

#### Alternatively

```bash
# (remote machine)
$ sudo nano ~/.ssh/authorized_keys
# (paste pub key here)

# (local machine)
$ cat -/.ssh/id_rsa.pub | ssh demo@192.168.1.1 "mkdir -p -/.ssh && chmod 700 -/.ssh $$ cat >> ~/.ssh/authorized_keys"
```

## Configure local SSH profile

Add to **~/.ssh/config**

```
Host GitHub
    Hostname github.com
    User git
    IdentityFile ~/.ssh/id_rsa

Host Raspberry
    Hostname 192.168.1.1
    User pi
    IdentityFile ~/.ssh/id_rsa
```

## Disable Password Login

```bash
nano /etc/ssh/sshd_config
```

Change the following settings:

* "ChallengeResponseAuthentication no"
* "PasswordAuthentication no"
* "UsePAM no"
* "PermitRootLogin no"

---

<br>

## SCP file transfer

```bash
# from local to remote
$ scp ~/file.txt user@remote:~/file.txt

# from remote to local 
$ scp user@remote:/file.txt ~/file.txt
```

<br>

## SSH Tunneling / Port Forwarding

```bash
$ ssh username@remote-server -L 5901:localhost:5900

$ ssh username@remote-server -R 5900:remote-server:5901
```

<br>
<br>

## Common Commands

```bash
# (Directories)
$ ls -lah
$ cd /etc/
$ cd ~/Documents
$ mkdir
$ rmdir
$ rm -r
$ pwd
$ cp
$ mv

# (Network)
$ nano /etc/dhcpcd.conf
$ nano /etc/netplan/
$ nmtui 
$ nmcli connection show
        [up/down] wlan0
        networking [on/off]
        status
        radio wifi [on/off]
$ ifconfig
$ ip a
$ curl ifconfig.io 

# Services
$ service ssh [status/start/stop/restart]
$ systemctl [status/start/stop/restart] sshd
$ /etc/init.d/ssh reload

# Alias
$nano ~/.bashrc
    alias la = "ls -a"

$ df -h
$ fdisk

# Users
$ adduser user
$ usermod -aG sudo user
$ deluser user
$ who
$ sudo su
$ passwd user

```
