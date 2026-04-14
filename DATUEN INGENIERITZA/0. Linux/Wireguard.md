
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
  #gateway 192.168.100.1
```

## Router
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
  address 192.168.100.1
  netmask 255.255.255.0
  #gateway 192.168.100.1


auto enp0s9
iface enp0s9 inet static
  address 192.168.100.1
  netmask 255.255.255.0
  #gateway 192.168.100.1
```

```
sudo nano /etc/sysctl.conf
net.ipv4.ip_forward=1
```

sudo sysctl -p

## VM1 eta VM2

```
sudo bash -c "wg genkey | tee /etc/wireguard/privatekey | wg pubkey > /etc/wireguard/publickey"
```

## VM1

/etc/wireguard/wg0.conf
```
[Interface]
Address = 10.0.0.1/24
ListenPort = 51820
PrivateKey = <PRIVKEY_VM1>

[Peer]
PublicKey = <PUBKEY_VM2>
AllowedIPs = 10.0.0.2/32
```
```
[Interface]
Address = 10.0.0.1/24
ListenPort = 51820
PrivateKey = sACU3VHZWDhh04rTwU2nS+O6lqRMVZlJSpS/aHdeQ2M=

[Peer]
PublicKey = m1jEjOhNV76U/MBiifRIsAraNGVhuNozbh3lfxaGOmU=
AllowedIPs = 10.0.0.2/32
```

## VM2
```
[Interface]
Address = 10.0.0.2/24
PrivateKey = <PRIVKEY_VM2>

[Peer]
PublicKey = <PUBKEY_VM1>
# Apunta a la IP del router en eth0, no a VM1 directamente
Endpoint = 192.168.100.1:51820
AllowedIPs = 10.0.0.1/32
PersistentKeepalive = 2
```
```
[Interface]
Address = 10.0.0.2/24
PrivateKey = kFcPQX/lVkVazC0eFLkbjdHnvNynhA/HaouPvqH8j2Y=

[Peer]
PublicKey = quViPiqInLTZEUYnJ8X20xe8b5KLPqe0lADnQ56AHiE=
# Apunta a la IP del router en eth0, no a VM1 directamente
Endpoint = 192.168.100.1:51820
AllowedIPs = 10.0.0.1/32
PersistentKeepalive = 25
```

## VM1 eta VM2 
```
sudo chmod 600 /etc/wireguard/privatekey /etc/wireguard/wg0.conf
sudo wg-quick up wg0
```
sudo wg show

## IP tables router
```
sudo apt install -y iptables
sudo apt install -y iptables iptables-persistent
```

```
# 1. Activar forwarding (por si acaso)
sudo sysctl -w net.ipv4.ip_forward=1

# 2. Limpiar reglas previas
sudo iptables -F
sudo iptables -t nat -F

# 3. FORWARD — permitir que los paquetes atraviesen el router
sudo iptables -A FORWARD -i enp0s9 -o enp0s8 -j ACCEPT
sudo iptables -A FORWARD -i enp0s8 -o enp0s9 -j ACCEPT

# 4. DNAT — redirigir :51820 entrante por enp0s9 hacia VM1
sudo iptables -t nat -A PREROUTING \
  -i enp0s9 -p udp --dport 51820 \
  -j DNAT --to-destination 192.168.100.10:51820

# 5. MASQUERADE — enmascarar origen para que VM1 no vea la IP duplicada
sudo iptables -t nat -A POSTROUTING \
  -o enp0s8 \
  -j MASQUERADE
```


```
ibai@vbox:~$ sudo iptables -t nat -D POSTROUTING -o enp0s8 -j MASQUERADE
ibai@vbox:~$ sudo iptables -t nat -A POSTROUTING -o enp0s8 -p udp --dport 51820 \
  -j SNAT --to-source 192.168.100.1
```

```
# Ver reglas NAT
sudo iptables -t nat -L -n -v --line-numbers

# Ver reglas FORWARD
sudo iptables -L FORWARD -n -v
```