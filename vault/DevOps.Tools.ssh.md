---
id: bdjaja4xyr9goopr7mxyigt
title: ssh
desc: ''
updated: 1662819597371
created: 1662816839907
---

Setting up ssh key to connect two machine

## 1. Create your own ssh key.

On your local machine generate your own key using sample command below.

```bash
ssh-keygen -t rsa -b 4096
```

Sample Output:

```bash
Generating public/private rsa key pair.
Enter file in which to save the key (/root/.ssh/id_rsa): 
Created directory '/root/.ssh'.
Enter passphrase (empty for no passphrase): 
Enter same passphrase again: 
Your identification has been saved in /root/.ssh/id_rsa.
Your public key has been saved in /root/.ssh/id_rsa.pub.
The key fingerprint is:
SHA256:eBq8ZvInWmvkS8Qvcd/utlzBCjkQH902vnnkANsFrps root@localhost.localdomain
The key's randomart image is:
+---[RSA 4096]----+
|        . .. ... |
|         o .o.+ .|
|        . .  *.o |
|     o . . ..o+ .|
|      B S + . o* |
|     ..O . + +o.o|
|    .oO . . E .. |
|     B+o.  o..   |
|    .o++   o=.   |
+----[SHA256]-----+
```

The key will be located on user home directory `/home/username/.ssh` and will be containing couple of file as shown below

```bash
[root@localhost .ssh]# ls -la
total 8
drwx------. 2 root root   38 Sep 10 09:36 .
dr-xr-x---. 4 root root  203 Sep 10 09:36 ..
-rw-------. 1 root root 3401 Sep 10 09:36 id_rsa
-rw-r--r--. 1 root root  752 Sep 10 09:36 id_rsa.pub
```

we will be needing `id_rsa.pub` that containing public key.

## 2. Copy your public key to the target machine.

you can add your public key to the remote machine using 2 option

1. copy using ssh-copy command

```bash
ssh-copy-id -i $HOME/.ssh/id_rsa.pub user@server.or.address.ip
```

2. Manual Way

login to your server/remote machine using existing solution (ssh or rdp)

create new directory and file at user directory either root or normal user

```
mkdir -p /home/user/.ssh/
# or
mkdir -p /root/.ssh/
```

then create a file named `authorized_keys` to contain your machine public key

```bash
vim /home/user/.ssh/authorized_keys
# or
vim /root/.ssh/
```

add your key to `authorized_keys`

```authorized_keys
# add command to give indication to who the key is belong, to make easier to manage
# pewpew key
ssh-rsa AAAAB3NzaC1------- this-is-your-key
# user1 key
ssh-rsa AAAAB3NzaC1------- another-user-key
```

You can now connect to the remote machine using ssh key

```bash
user@server.or.address.ip
# or
root@server.or.address.ip
```

## Extra steps, Disable ssh password login

We need to modify sshd configuration to disable ssh login using password so only user with ssh public kay that aproved/added can login to remote machine

```bash
sudo vim /etc/ssh/sshd_config
```

find and change these value

option | value |
---------|----------
 ChallengeResponseAuthentication | no
 PasswordAuthentication | no
 UsePAM  | no

after change and save the sshd config file restart sshd service

```bash
sudo systemctl restart sshd
sudo systemctl status sshd
```