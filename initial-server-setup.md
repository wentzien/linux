# Initial Server Setup

```
$ ssh root@your_server_ip
```

## Create new admin user
Add user and granting administrative privileges:
```
$ adduser wntzn
$ usermod -aG sudo wntzn
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

