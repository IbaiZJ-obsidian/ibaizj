
```
sudo apt update && sudo apt install -y wireguard
```

![[Wireguard.excalidraw]]


## VM1 eta VM2

```
# This file describes the network interfaces available on your system
# and how to activate them. For more information, see interfaces(5).

source /etc/network/interfaces.d/*

# The loopback network interface
auto lo
iface lo inet loopback

# The primary network interface
allow-hotplug enp0s3
iface enp0s3 inet dhcp
# This is an autoconfigured IPv6 interface
iface enp0s3 inet6 auto


# -> router
auto enp0s8
iface enp0s8 inet static
  address 192.168.100.10
  netmask 255.255.255.0
  gateway 192.168.100.1
```

