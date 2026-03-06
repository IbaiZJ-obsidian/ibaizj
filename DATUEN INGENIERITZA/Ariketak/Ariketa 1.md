
![[4 Datuen Ingeniaritza - 2.2.2 Datu masiboentzako azpiegitura oinarriak - Oinarrizko kontzeptuak - Ariketa 1.pdf#page=2]]

![[Pasted image 20260306220729.png]]
# 




































# Debian makinaren sare konfigurazioa

## ip a

```
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
    inet6 ::1/128 scope host noprefixroute
       valid_lft forever preferred_lft forever
2: enp0s3: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc fq_codel state UP group default qlen 1000
    link/ether 08:00:27:8a:88:c9 brd ff:ff:ff:ff:ff:ff
    altname enx0800278a88c9
    inet 10.0.2.15/24 brd 10.0.2.255 scope global dynamic noprefixroute enp0s3
       valid_lft 85394sec preferred_lft 74594sec
    inet6 fd17:625c:f037:2:a00:27ff:fe8a:88c9/64 scope global dynamic mngtmpaddr proto kernel_ra
       valid_lft 86116sec preferred_lft 14116sec
    inet6 fd17:625c:f037:2:80e:7b11:1252:58b3/64 scope global dynamic mngtmpaddr noprefixroute
       valid_lft 86116sec preferred_lft 14116sec
    inet6 fe80::49e:224e:e3fb:fec1/64 scope link
       valid_lft forever preferred_lft forever
3: enp0s8: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc fq_codel state UP group default qlen 1000
    link/ether 00:01:02:03:04:05 brd ff:ff:ff:ff:ff:ff
    altname enx000102030405
    inet 192.168.8.2/23 brd 192.168.9.255 scope global enp0s8
       valid_lft forever preferred_lft forever
    inet6 fe80::201:2ff:fe03:405/64 scope link proto kernel_ll
       valid_lft forever preferred_lft forever
```

## /etc/network/interfaces

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

# Red interna kudeatzailea

auto enp0s8
iface enp0s8 inet static
        address 192.168.8.2/23
```
# Proxmox makinaren sare konfigurazioa

## ip a

```
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
    inet6 ::1/128 scope host noprefixroute
       valid_lft forever preferred_lft forever
2: nic0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc fq_codel state UP group default qlen 1000
    link/ether 08:00:27:af:50:84 brd ff:ff:ff:ff:ff:ff
    altname enx080027af5084
    inet 10.0.2.15/24 brd 10.0.2.255 scope global dynamic nic0
       valid_lft 85298sec preferred_lft 85298sec
    inet6 fd17:625c:f037:2:a00:27ff:feaf:5084/64 scope global dynamic mngtmpaddr proto kernel_ra
       valid_lft 86202sec preferred_lft 14202sec
    inet6 fe80::a00:27ff:feaf:5084/64 scope link proto kernel_ll
       valid_lft forever preferred_lft forever
3: nic1: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc fq_codel master vmbr0 state UP group default qlen 1000
    link/ether 00:01:02:03:04:06 brd ff:ff:ff:ff:ff:ff
    altname enx000102030406
4: nic2: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc fq_codel master vmbr1 state UP group default qlen 1000
    link/ether 00:01:02:03:04:07 brd ff:ff:ff:ff:ff:ff
    altname enx000102030407
5: vmbr0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc noqueue state UP group default qlen 1000
    link/ether 00:01:02:03:04:06 brd ff:ff:ff:ff:ff:ff
    inet 192.168.8.1/23 scope global vmbr0
       valid_lft forever preferred_lft forever
    inet6 fe80::201:2ff:fe03:406/64 scope link proto kernel_ll
       valid_lft forever preferred_lft forever
6: vmbr1: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc noqueue state UP group default qlen 1000
    link/ether 00:01:02:03:04:07 brd ff:ff:ff:ff:ff:ff
    inet 192.168.12.1/23 scope global vmbr1
       valid_lft forever preferred_lft forever
    inet6 fe80::201:2ff:fe03:407/64 scope link proto kernel_ll
       valid_lft forever preferred_lft forever
```

## /etc/network/interfaces

```
auto lo
iface lo inet loopback

auto nic0
allow-hotplug nic0
iface nic0 inet dhcp

iface nic1 inet manual

auto vmbr0
iface vmbr0 inet static
        address 192.168.8.1/23
        bridge-ports nic1
        bridge-stp off
        bridge-fd 0


iface nic2 inet manual

auto vmbr1
iface vmbr1 inet static
        address 192.168.12.1/23
        gateway 192.168.13.254
        bridge-ports nic2
        bridge-stp off
        bridge-fd 0


source /etc/network/interfaces.d/*
```
# Router makinaren sare konfigurazioa

```
nano /etc/hostname
```

```
nano /etc/hosts
```

## ip a

```
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
    inet6 ::1/128 scope host noprefixroute
       valid_lft forever preferred_lft forever
2: enp0s3: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc fq_codel state UP group default qlen 1000
    link/ether 08:00:27:a1:86:da brd ff:ff:ff:ff:ff:ff
    altname enx080027a186da
    inet 10.0.2.15/24 brd 10.0.2.255 scope global dynamic noprefixroute enp0s3
       valid_lft 86012sec preferred_lft 75212sec
    inet6 fd17:625c:f037:2:5684:5e8:a024:cefc/64 scope global dynamic mngtmpaddr noprefixroute
       valid_lft 86248sec preferred_lft 14248sec
    inet6 fe80::3f03:7736:4130:5bb6/64 scope link
       valid_lft forever preferred_lft forever
3: enp0s8: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc fq_codel state UP group default qlen 1000
    link/ether 10:01:02:03:04:05 brd ff:ff:ff:ff:ff:ff
    altname enx100102030405
    inet 192.168.13.254/23 brd 192.168.13.255 scope global enp0s8
       valid_lft forever preferred_lft forever
    inet6 fe80::1201:2ff:fe03:405/64 scope link proto kernel_ll
       valid_lft forever preferred_lft forever
```

## /etc/network/interfaces

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
        post-up nft add table ip nat 2>/dev/null || true
        post-up nft add chain ip nat POSTROUTING '{ type nat hook postrouting priority srcnat; policy accept; }' 2>/dev/null || true
        post-up nft add rule ip nat POSTROUTING oifname "enp0s3" masquerade || true
        pre-down nft delete rule ip nat POSTROUTING oifname "enp0s3" masquerade || true


# Red interna kudeatzailea

allow-hotplug enp0s8
iface enp0s8 inet static
        address 192.168.13.254/23
```


```
nano /usr/lib/sysctl.d/50-router.conf
```

```
net.ipv4.ip_forward = 1
```