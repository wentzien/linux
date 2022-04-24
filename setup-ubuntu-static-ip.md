# Setup static ip in vlan

## Identify ethernet interfaces

```
ip link
```

```
ip a
```

## Configure netplan configuration files

```
sudo vim /etc/netplan/02-vlan.yaml
```

```
network:
   version: 2
   renderer: networkd
   ethernets:
      eth1:
        dhcp4: no
        addresses:
        - 10.0.x.x/16
```

```
sudo netplan generate
```

```
sudo netplan apply
```