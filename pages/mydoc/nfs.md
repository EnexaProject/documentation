---
title: NFS
keywords: ENEXA Documentation
sidebar: mydoc_sidebar
toc: false
permalink: nfs.html
folder: mydoc
---

### 1. Verify NFS Server Installation
   Ensure that the NFS server packages are installed on your server:


```sh
sudo apt update
sudo apt install nfs-kernel-server
```


### 2. Configure NFS Exports
Ensure your `/etc/exports` file is correctly configured. Your entry looks correct:


```
/data/nfs/kubedata  *(rw,sync,no_subtree_check)
```


### 3. Apply Export Changes
After editing `/etc/exports`, you need to apply the changes:


```sh
sudo exportfs -ra
```


### 4. Check NFS Server Status
Ensure the NFS server is running:


```sh
sudo systemctl status nfs-server
```


If it's not running, start it:


```sh
sudo systemctl start nfs-server
```


And enable it to start on boot:


```sh
sudo systemctl enable nfs-server
```


### 5. Configure Firewall
Ensure that your firewall allows NFS traffic. You need to open the following ports for NFS:


- 111 (rpcbind/sunrpc)
- 2049 (nfs)
- 20048 (mountd, if used)


You can do this using `ufw` (Uncomplicated Firewall) or `iptables`.


For `ufw`, use:


```sh
sudo ufw allow 111
sudo ufw allow 2049
sudo ufw allow 20048
sudo ufw reload
```


For `iptables`, use:


```sh
sudo iptables -A INPUT -p tcp --dport 111 -j ACCEPT
sudo iptables -A INPUT -p udp --dport 111 -j ACCEPT
sudo iptables -A INPUT -p tcp --dport 2049 -j ACCEPT
sudo iptables -A INPUT -p udp --dport 2049 -j ACCEPT
sudo iptables -A INPUT -p tcp --dport 20048 -j ACCEPT
sudo iptables -A INPUT -p udp --dport 20048 -j ACCEPT
sudo iptables-save | sudo tee /etc/iptables/rules.v4
```


### 6. Check Network Configuration
Ensure that there are no network issues preventing access:


- Verify that the server and client can ping each other.
- Check for any network policies or firewalls that might be blocking the traffic.


### 7. Client Configuration
On the client side, ensure the NFS client package is installed:


```sh
sudo apt update
sudo apt install nfs-common
```


Then, try to mount the NFS share:


```sh
sudo mount -t nfs <server-ip>:/data/nfs/kubedata /mnt
```


Replace `<server-ip>` with the actual IP address of your NFS server.


### 8. Verify Mount
Check if the NFS share is mounted correctly:


```sh
df -h /mnt
```
