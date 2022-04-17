# Initial Server Setup

```
$ ssh root@<your_server_ip>
```

## Create new admin user
Add user and granting administrative privileges:
```
$ adduser <username>
$ usermod -aG sudo <username>
```

## Basic firewall
ufw is inclueded in the basic installation, but could be installed with:
```
$ sudo apt-get install ufw gufw
```

overview about current application filters:
```
$ ufw app list
```

allow SSH connections
```
$ sudo ufw allow OpenSSH
```

check status and if necessary activate:
```
$ sudo ufw status
$ sudo ufw enable
```

## Disable root ssh login

open and edit sshd config
```
vim /etc/ssh/sshd_config
```

modify following line
```
PermitRootLogin yes
```

disable root login
```
PermitRootLogin no
```

add whitelist for created user
```
AllowUsers <username>
```

restart ssh service
```
service ssh restart
```