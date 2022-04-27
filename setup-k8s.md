# Setup K8s

## Allow traffic from subnet and calico network

master and worker:

```
sudo ufw allow from 10.0.0.0/16
sudo ufw allow from 192.168.0.0/16
sudo ufw allow 10250/tcp
```

just on master:

```
sudo ufw allow 6443
```