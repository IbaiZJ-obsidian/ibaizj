
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





```
Hola, me ayudas a configurar wireguard entre 2 vm de vbox que no tienen conectividad entre ellos? quiero que uno sea el servidor y el otro el cliente, dame tambien tips o dime como tendria que hacer para añadir mas clientes a la vpn, estoy usando debian en todas las maquinas

14 abr

¡Buenas! Vamos a ello. Primero el diagrama de cómo va a quedar la arquitectura, y luego la guía completa.

La idea clave: las VMs no se ven entre sí por NAT, así que les añadimos un **Host-Only Adapter** como canal de transporte para el túnel WireGuard.

---

## 1. Preparar VirtualBox

En cada VM necesitas **dos adaptadores de red**:

- **Adaptador 1** → NAT (para que tengan internet y puedan instalar paquetes)
- **Adaptador 2** → Host-Only Adapter (para que se vean entre sí)

En VirtualBox: `Settings → Network → Adapter 2 → Enable → Host-only Adapter → vboxnet0`

Si no tienes la red Host-Only creada: `File → Host Network Manager → Create` (rango típico `192.168.56.0/24`).

Asigna IPs estáticas en el adaptador host-only de cada VM editando `/etc/network/interfaces` o con `nmcli`. Por ejemplo:

**VM1 (servidor):** `192.168.56.10/24` **VM2 (cliente):** `192.168.56.20/24`

Verifica conectividad antes de seguir: `ping 192.168.56.20` desde VM1.

---

## 2. Instalar WireGuard (en ambas VMs)

bash

```bash
sudo apt update && sudo apt install -y wireguard
```

---

## 3. Generar claves (en cada VM por separado)

bash

```bash
# En VM1
wg genkey | tee /etc/wireguard/privatekey | wg pubkey > /etc/wireguard/publickey
cat /etc/wireguard/privatekey   # guarda esto
cat /etc/wireguard/publickey    # guarda esto también

# Lo mismo en VM2
wg genkey | tee /etc/wireguard/privatekey | wg pubkey > /etc/wireguard/publickey
```

---

## 4. Configurar el servidor — VM1

Crea `/etc/wireguard/wg0.conf`:

ini

```ini
[Interface]
Address = 10.0.0.1/24
ListenPort = 51820
PrivateKey = <CLAVE_PRIVADA_VM1>

[Peer]
# VM2 - Cliente
PublicKey = <CLAVE_PUBLICA_VM2>
AllowedIPs = 10.0.0.2/32
```

---

## 5. Configurar el cliente — VM2

Crea `/etc/wireguard/wg0.conf`:

ini

```ini
[Interface]
Address = 10.0.0.2/24
PrivateKey = <CLAVE_PRIVADA_VM2>

[Peer]
# VM1 - Servidor
PublicKey = <CLAVE_PUBLICA_VM1>
Endpoint = 192.168.56.10:51820
AllowedIPs = 10.0.0.0/24
PersistentKeepalive = 25
```

`PersistentKeepalive` es importante: como el cliente está detrás de NAT, mantiene el túnel vivo enviando un keepalive cada 25 segundos.

---

## 6. Levantar la VPN

bash

```bash
# En ambas VMs
sudo chmod 600 /etc/wireguard/privatekey /etc/wireguard/wg0.conf
sudo wg-quick up wg0

# Para que arranque automáticamente al boot
sudo systemctl enable wg-quick@wg0
```

Verifica que el túnel esté activo:

bash

```bash
sudo wg show
```

Deberías ver el peer listado. Prueba conectividad:

bash

```bash
# Desde VM2
ping 10.0.0.1
```

---

## 7. Añadir más clientes

Este es uno de los puntos fuertes de WireGuard: añadir un cliente nuevo es trivial.

**Para cada nuevo cliente (ej. VM3):**

1. Instala WireGuard en VM3 y genera sus claves.
2. Añade un nuevo bloque `[Peer]` en el `wg0.conf` del servidor (VM1):

ini

```ini
[Peer]
# VM3 - Nuevo cliente
PublicKey = <CLAVE_PUBLICA_VM3>
AllowedIPs = 10.0.0.3/32
```

3. Recarga la configuración del servidor **sin tirar el túnel**:

bash

```bash
sudo wg addconf wg0 <(wg-quick strip wg0)
# O más simple:
sudo wg set wg0 peer <CLAVE_PUBLICA_VM3> allowed-ips 10.0.0.3/32
```

4. El `wg0.conf` de VM3 sería idéntico al de VM2 pero con `Address = 10.0.0.3/24` y apuntando al mismo `Endpoint` del servidor.

**Convención de IPs:** asigna `10.0.0.X/32` por cliente en el servidor y `10.0.0.X/24` en el propio cliente. El `/32` en el servidor evita que un cliente enrute tráfico de otros peers sin permiso explícito.

---

## Troubleshooting rápido

|Síntoma|Causa probable|
|---|---|
|`ping` no llega|Firewall bloqueando UDP 51820 — comprueba `ufw allow 51820/udp`|
|`wg show` no muestra handshake|Endpoint incorrecto o IP Host-Only mal configurada|
|Túnel cae tras inactividad|Añade `PersistentKeepalive = 25` en el cliente|
|Error al levantar `wg0`|Permisos del fichero — `chmod 600 /etc/wireguard/wg0.conf`|

quiero darle una vuelta a la configuracion, quiero configurar nats y que no esten conectados directamente las 2 maquinas, para eso quiero tener una 3 maquina en medio que haga de router, como lo hago?

14 abr

Buena idea, es una topología más realista (simula que los peers están en redes distintas sin visibilidad directa). Primero el diagrama de cómo va a quedar:

La respuesta de Claude fue interrumpida

quiero que las 2 maquinas tambien tengan la misma red para tener que hacer alguna cosa con los nat y poder arreglarlo

14 abr

Buena idea, así la práctica tiene más "chicha" — simulas un conflicto de IPs real como el que te encuentras en empresas cuando dos redes tienen el mismo rango. El router tiene que hacer NAT doble para que todo funcione.

El problema central: ambas VMs tienen `192.168.100.10` — el router no puede distinguirlas por IP, así que necesita interfaces separadas (una red interna distinta para cada lado) y hace NAT en ambos sentidos para presentarle a cada VM una IP diferente de la otra.

---

## 1. Configurar VirtualBox

**VM1 y VM2** — cada una con **dos adaptadores**:

- Adaptador 1: `Internal Network` → nombre `intnet1` (VM1) / `intnet2` (VM2)
- Adaptador 2: NAT (para instalar paquetes, puedes quitarlo después)

**VM3 (router)** — con **tres adaptadores**:

- Adaptador 1: `Internal Network` → `intnet1`
- Adaptador 2: `Internal Network` → `intnet2`
- Adaptador 3: NAT (internet, para instalar paquetes)

---

## 2. Asignar IPs estáticas

**VM1** `/etc/network/interfaces`:

```
auto eth0
iface eth0 inet static
  address 192.168.100.10
  netmask 255.255.255.0
  gateway 192.168.100.1
```

**VM2** — exactamente igual (mismo rango, mismo gateway, misma IP incluso):

```
auto eth0
iface eth0 inet static
  address 192.168.100.10
  netmask 255.255.255.0
  gateway 192.168.100.1
```

**VM3** `/etc/network/interfaces`:

```
# hacia VM1
auto eth0
iface eth0 inet static
  address 192.168.100.1
  netmask 255.255.255.0

# hacia VM2
auto eth1
iface eth1 inet static
  address 192.168.100.1
  netmask 255.255.255.0
```

Sí, el router tiene la misma IP en ambas interfaces — pero en interfaces físicas distintas, así que no hay conflicto. Cada `intnet` es una red aislada.

---

## 3. Activar ip_forward en VM3

bash

```bash
echo "net.ipv4.ip_forward=1" >> /etc/sysctl.conf
sysctl -p
```

---

## 4. Instalar WireGuard y generar claves

bash

```bash
# En VM1 y VM2
sudo apt update && sudo apt install -y wireguard

# En cada una, generar claves
wg genkey | tee /etc/wireguard/privatekey | wg pubkey > /etc/wireguard/publickey
```

---

## 5. Configurar WireGuard

**VM1** `/etc/wireguard/wg0.conf`:

ini

```ini
[Interface]
Address = 10.0.0.1/24
ListenPort = 51820
PrivateKey = <PRIVKEY_VM1>

[Peer]
PublicKey = <PUBKEY_VM2>
AllowedIPs = 10.0.0.2/32
```

**VM2** `/etc/wireguard/wg0.conf`:

ini

```ini
[Interface]
Address = 10.0.0.2/24
PrivateKey = <PRIVKEY_VM2>

[Peer]
PublicKey = <PUBKEY_VM1>
# Apunta a la IP del router en eth0, no a VM1 directamente
Endpoint = 192.168.100.1:51820
AllowedIPs = 10.0.0.1/32
PersistentKeepalive = 25
```

---

## 6. Reglas iptables en VM3 — el núcleo del ejercicio

Aquí es donde está toda la chicha. El router tiene que:

1. Redirigir el UDP 51820 entrante por `eth1` (desde VM2) hacia VM1
2. Hacer SNAT para que el tráfico de cada lado tenga una IP no ambigua

bash

```bash
# Limpiar reglas previas
iptables -F
iptables -t nat -F

# DNAT: cuando llega tráfico UDP al puerto 51820 por eth1 (lado VM2),
# lo redirigimos a VM1 en eth0
iptables -t nat -A PREROUTING -i eth1 -p udp --dport 51820 \
  -j DNAT --to-destination 192.168.100.10:51820

# SNAT lado VM2→VM1: el router se enmascara con su IP de eth0
# para que VM1 vea el tráfico como si viniera del router (no de 192.168.100.10 de VM2)
iptables -t nat -A POSTROUTING -o eth0 -p udp --dport 51820 \
  -j MASQUERADE

# SNAT lado VM1→VM2: igual al revés
iptables -t nat -A POSTROUTING -o eth1 \
  -j MASQUERADE

# Permitir el forward entre interfaces
iptables -A FORWARD -i eth1 -o eth0 -j ACCEPT
iptables -A FORWARD -i eth0 -o eth1 -j ACCEPT
```

El `MASQUERADE` es lo que resuelve el conflicto: cuando VM2 manda un paquete desde `192.168.100.10`, el router lo reescribe como si viniera de `192.168.100.1` (su propia IP en eth0) antes de mandarlo a VM1. Así VM1 nunca ve dos paquetes con la misma IP origen.

Para hacer las reglas persistentes:

bash

```bash
apt install -y iptables-persistent
netfilter-persistent save
```

---

## 7. Levantar y verificar

bash

```bash
# En VM1 y VM2
sudo chmod 600 /etc/wireguard/wg0.conf /etc/wireguard/privatekey
sudo wg-quick up wg0
sudo systemctl enable wg-quick@wg0
```

Verifica el handshake:

bash

```bash
# En VM1 o VM2
sudo wg show
```

Prueba de conectividad por el túnel:

bash

```bash
# Desde VM2, ping a la IP WireGuard de VM1
ping 10.0.0.1

# Desde VM1
ping 10.0.0.2
```

Si el handshake aparece pero el ping no responde, el problema casi siempre es una regla FORWARD que falta o el `ip_forward` que no está activo. Comprueba con `sysctl net.ipv4.ip_forward` — tiene que devolver `1`.

```
sudo apt update && sudo apt install -y wireguard
```

Wireguard.excalidraw VM1 eta VM2

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

Router

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


auto enp0s9
iface enp0s9 inet static
  address 192.168.100.10
  netmask 255.255.255.0
  gateway 192.168.100.1
```

```
sudo nano /etc/sysctl.conf
net.ipv4.ip_forward=1
```

sudo sysctl -p VM1 eta VM2

```
sudo bash -c "wg genkey | tee /etc/wireguard/privatekey | wg pubkey > /etc/wireguard/publickey"
```

VM1 /etc/wireguard/wg0.conf

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

VM2

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

he hecho esto de momento, como continuo? como se si tengo conectividad entre las vm y el "router"? ibai@vbox:~$ ping 192.168.100.1 PING 192.168.100.1 (192.168.100.1) 56(84) bytes of data. ^C --- 192.168.100.1 ping statistics --- 2 packets transmitted, 0 received, 100% packet loss, time 1029ms

Mostrar más

1:02

Veo el problema — el ping al router no llega. Antes de meternos con WireGuard hay que asegurarse de que la conectividad básica funciona. Vamos por pasos.

## 1. Diagnóstico: ¿la interfaz enp0s8 está levantada?

En VM1 (o VM2):

bash

```bash
ip addr show enp0s8
ip link show enp0s8
```

Si aparece `DOWN` o no tiene IP asignada:

bash

```bash
sudo ifup enp0s8
# o
sudo systemctl restart networking
```

## 2. Problema probable — el `gateway` en VM1/VM2

Mira tu config de VM1/VM2:

```
auto enp0s8
iface enp0s8 inet static
  address 192.168.100.10
  netmask 255.255.255.0
  gateway 192.168.100.1   # <-- ESTO puede causar conflicto
```

La línea `gateway` en `enp0s8` compite con el default gateway de `enp0s3` (el NAT). **Quítala de enp0s8** — el gateway por defecto lo gestiona el NAT, no la red interna:

```
auto enp0s8
iface enp0s8 inet static
  address 192.168.100.10
  netmask 255.255.255.0
```

Aplica los cambios:

bash

```bash
sudo ifdown enp0s8 && sudo ifup enp0s8
```

## 3. Mismo problema en el router — y otro más gordo

Tu config del router tiene esto:

```
auto enp0s8
  address 192.168.100.10   # <-- debería ser 192.168.100.1
  gateway 192.168.100.1    # <-- gateway de sí mismo, no tiene sentido

auto enp0s9
  address 192.168.100.10   # <-- debería ser 192.168.100.1
  gateway 192.168.100.1    # <-- idem
```

El router tiene que ser el **gateway** (`192.168.100.1`), no un cliente. Corrígelo:

```
# -> intnet1 (hacia VM1)
auto enp0s8
iface enp0s8 inet static
  address 192.168.100.1
  netmask 255.255.255.0

# -> intnet2 (hacia VM2)
auto enp0s9
iface enp0s9 inet static
  address 192.168.100.1
  netmask 255.255.255.0
```

Sin `gateway` en ninguna de las dos — el gateway del router es `enp0s3` (NAT).

Aplica:

bash

```bash
sudo systemctl restart networking
```

## 4. Verificar conectividad paso a paso

Después de corregir, comprueba en este orden:

bash

```bash
# 1. Ver que las interfaces tienen la IP correcta
ip addr

# 2. Ver la tabla de rutas — no debería haber dos default gw
ip route

# 3. Ping al router desde VM1
ping -c3 192.168.100.1

# 4. Ping al router desde VM2
ping -c3 192.168.100.1

# 5. Desde el router, ping a ambas VMs
ping -c3 192.168.100.10   # con enp0s8
ping -c3 192.168.100.10   # con enp0s9 -- esto lo haremos con -I para especificar interfaz
ping -c3 -I enp0s8 192.168.100.10
ping -c3 -I enp0s9 192.168.100.10
```

Dime qué devuelve `ip addr` e `ip route` en cada máquina y lo vemos.

desde e router si, desde las vm no

ibai@vbox:~$ ip a 1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN group default qlen 1000 link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00 inet 127.0.0.1/8 scope host lo valid_lft forever preferred_lft forever inet6 ::1/128 scope host noprefixroute valid_lft forever preferred_lft forever 2: enp0s3: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc fq_codel state UP group default qlen 1000 link/ether 08:00:27:6c:c1:b7 brd ff:ff:ff:ff:ff:ff altname enx0800276cc1b7 inet 10.0.2.15/24 brd 10.0.2.255 scope global dynamic noprefixroute enp0s3 valid_lft 82714sec preferred_lft 71914sec inet6 fd17:625c:f037:2:a00:27ff:fe6c:c1b7/64 scope global dynamic mngtmpaddr proto kernel_ra valid_lft 86308sec preferred_lft 14308sec inet6 fd17:625c:f037:2:3bf7:955d:a7c2:7716/64 scope global dynamic mngtmpaddr noprefixroute valid_lft 86308sec preferred_lft 14308sec inet6 fe80::e48c:cfce:7453:1d44/64 scope link valid_lft forever preferred_lft forever 3: enp0s8: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc fq_codel state UP group default qlen 1000 link/ether 08:00:27:d5:23:ef brd ff:ff:ff:ff:ff:ff altname enx080027d523ef inet 192.168.100.10/24 brd 192.168.100.255 scope global enp0s8 valid_lft forever preferred_lft forever inet6 fe80::a00:27ff:fed5:23ef/64 scope link proto kernel_ll valid_lft forever preferred_lft forever ibai@vbox:~$ ip route default via 10.0.2.2 dev enp0s3 proto dhcp src 10.0.2.15 metric 1002 10.0.2.0/24 dev enp0s3 proto dhcp scope link src 10.0.2.15 metric 1002 192.168.100.0/24 dev enp0s8 proto kernel scope link src 192.168.100.10 ibai@vbox:~$ ping 192.168.100.1 PING 192.168.100.1 (192.168.100.1) 56(84) bytes of data. ^C --- 192.168.100.1 ping statistics --- 2 packets transmitted, 0 received, 100% packet loss, time 1003ms

Mostrar más

1:06

Las IPs están bien asignadas, la ruta existe (`192.168.100.0/24 dev enp0s8`), pero los paquetes no llegan. Casi seguro es un problema de VirtualBox — las VMs no están en la misma `intnet`.

## 1. Verificar que están en la misma Internal Network

En VirtualBox, entra en la config de cada VM → **Network → Adapter 2** y comprueba:

- VM1: `Internal Network` → nombre exacto, ej. `intnet1`
- Router enp0s8: mismo nombre exacto → `intnet1`

El nombre es case-sensitive y tiene que ser **idéntico** carácter a carácter. Si uno tiene `intnet1` y el otro `Intnet1` no se ven.

## 2. Verificar que el adaptador está en modo Internal Network (no Host-Only)

A veces se confunde. Tiene que ser exactamente **Internal Network**, no Host-Only Adapter.

## 3. Comprobar que la VM del router tiene las IPs correctas

Desde el router:

bash

```bash
ip addr show enp0s8
ip addr show enp0s9
```

Dime qué IPs tiene. Si tiene `192.168.100.10` en lugar de `192.168.100.1`, el problema es la config que te dije antes — el router se configuró como cliente en vez de como gateway.

## 4. Si el nombre de intnet es correcto, hacer tcpdump para ver si llegan los paquetes

En el router, mientras haces ping desde VM1:

bash

```bash
# En el router
sudo tcpdump -i enp0s8 icmp

# En VM1 simultáneamente
ping 192.168.100.1
```

Hay dos casos:

- **tcpdump no muestra nada** → problema de VirtualBox, los adaptadores no están en la misma intnet
- **tcpdump muestra los pings pero no hay respuesta** → problema de firewall o config del router

¿Qué nombre de Internal Network pusiste en cada adaptador?

bale, ya tengo conectividad, ahora que hago?

1:09

Perfecto! Ahora toca levantar WireGuard y configurar las iptables del router. Sigue este orden:

## 1. Levantar WireGuard en VM1 y VM2

bash

```bash
# En ambas VMs
sudo chmod 600 /etc/wireguard/privatekey /etc/wireguard/wg0.conf
sudo wg-quick up wg0
```

Verifica que levantó bien:

bash

```bash
sudo wg show
```

En VM1 deberías ver algo así:

```
interface: wg0
  public key: quViPiqI...
  private key: (hidden)
  listening port: 51820

peer: m1jEjOhN...
  allowed ips: 10.0.0.2/32
```

En VM2 igual pero sin `latest handshake` todavía — el handshake no ocurre hasta que VM2 intente conectar.

## 2. Reglas iptables en el router

bash

```bash
# Limpiar reglas previas
sudo iptables -F
sudo iptables -t nat -F

# Permitir forwarding entre interfaces
sudo iptables -A FORWARD -i enp0s8 -o enp0s9 -j ACCEPT
sudo iptables -A FORWARD -i enp0s9 -o enp0s8 -j ACCEPT

# DNAT: tráfico UDP 51820 que llega por enp0s9 (lado VM2) → redirigir a VM1
sudo iptables -t nat -A PREROUTING -i enp0s9 -p udp --dport 51820 \
  -j DNAT --to-destination 192.168.100.10:51820

# MASQUERADE: reescribir IP origen para evitar el conflicto de IPs iguales
sudo iptables -t nat -A POSTROUTING -o enp0s8 -j MASQUERADE
sudo iptables -t nat -A POSTROUTING -o enp0s9 -j MASQUERADE
```

Verifica que las reglas están:

bash

```bash
sudo iptables -t nat -L -n -v
sudo iptables -L FORWARD -n -v
```

## 3. Probar el túnel

Desde VM2:

bash

```bash
ping -c3 10.0.0.1
```

Y comprueba que el handshake se produjo:

bash

```bash
# En VM1 o VM2
sudo wg show
```

Deberías ver `latest handshake` con un timestamp reciente.

## 4. Si el ping no llega — debug rápido

bash

```bash
# En el router, mientras haces ping desde VM2
sudo tcpdump -i enp0s9 udp port 51820
# ¿Llegan los paquetes WireGuard al router?

sudo tcpdump -i enp0s8 udp port 51820  
# ¿Los reenvía hacia VM1?
```

Dime qué devuelve `sudo wg show` en cada VM después de intentar el ping.

enseñame a hacer el nateo

1:11

Claro, antes de meter los comandos te explico qué va a pasar exactamente para que tenga sentido lo que escribes.

## El problema

VM1 y VM2 tienen la misma IP (`192.168.100.10`). Cuando VM2 manda un paquete UDP al router con destino `:51820`, el router tiene que:

1. **DNAT** — cambiar el destino del paquete: en vez de llegar a `192.168.100.1:51820` (el propio router), redirigirlo a `192.168.100.10:51820` (VM1), pero saliendo por `enp0s8`
2. **MASQUERADE** — cambiar el origen del paquete: en vez de que VM1 vea que viene de `192.168.100.10` (que es su propia IP), que vea que viene de `192.168.100.1` (el router)

Sin el MASQUERADE, VM1 recibiría un paquete que parece venir de sí misma y lo descartaría.

El flujo completo:

```
VM2 (192.168.100.10)
  → router enp0s9 [PREROUTING: DNAT destino → 192.168.100.10:51820]
  → router enp0s8 [POSTROUTING: MASQUERADE origen → 192.168.100.1]
  → VM1 ve: paquete de 192.168.100.1 para ella en :51820 ✓
```

---

## Las reglas

bash

```bash
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

---

## Verificar que las reglas están bien

bash

```bash
# Ver reglas NAT
sudo iptables -t nat -L -n -v --line-numbers

# Ver reglas FORWARD
sudo iptables -L FORWARD -n -v
```

Deberías ver algo así en la tabla nat:

```
Chain PREROUTING
1  DNAT  udp  --  enp0s9  *  0.0.0.0/0  0.0.0.0/0  udp dpt:51820  to:192.168.100.10:51820

Chain POSTROUTING
1  MASQUERADE  all  --  *  enp0s8  0.0.0.0/0  0.0.0.0/0
```

---

## Probar

bash

```bash
# Levantar WireGuard si no lo has hecho ya
sudo wg-quick up wg0   # en VM1 y VM2

# Desde VM2
ping -c3 10.0.0.1

# Ver si hubo handshake
sudo wg show
```

Dime qué ves en `sudo wg show` después del ping.

sudo iptables -F sudo: iptables: command not found

1:12

Normal en Debian minimal. Instálalo:

bash

```bash
sudo apt install -y iptables
```

Y ya puedes continuar con las reglas. Si quieres también instala `iptables-persistent` de una vez para que las reglas sobrevivan al reinicio:

bash

```bash
sudo apt install -y iptables iptables-persistent
```

Durante la instalación te pregunta si quieres guardar las reglas actuales — di **No** por ahora, las guardamos después de configurarlas bien.

router

ibai@vbox:~$ sudo iptables -t nat -L -n -v --line-numbers Chain PREROUTING (policy ACCEPT 0 packets, 0 bytes) num pkts bytes target prot opt in out source destination 1 1 176 DNAT udp -- enp0s9 * 0.0.0.0/0 0.0.0.0/0 udp dpt:51820 to:192.168.100.10:51820 Chain INPUT (policy ACCEPT 0 packets, 0 bytes) num pkts bytes target prot opt in out source destination Chain OUTPUT (policy ACCEPT 0 packets, 0 bytes) num pkts bytes target prot opt in out source destination Chain POSTROUTING (policy ACCEPT 0 packets, 0 bytes) num pkts bytes target prot opt in out source destination 1 0 0 MASQUERADE all -- * enp0s8 0.0.0.0/0 0.0.0.0/0 ibai@vbox:~$ sudo iptables -L FORWARD -n -v Chain FORWARD (policy ACCEPT 0 packets, 0 bytes) pkts bytes target prot opt in out source destination 6 1056 ACCEPT all -- enp0s9 enp0s8 0.0.0.0/0 0.0.0.0/0 0 0 ACCEPT all -- enp0s8 enp0s9 0.0.0.0/0 0.0.0.0/0 ibai@vbox:~$

vm2 ibai@vbox:~$ ip a 1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN group default qlen 1000 link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00 inet 127.0.0.1/8 scope host lo valid_lft forever preferred_lft forever inet6 ::1/128 scope host noprefixroute valid_lft forever preferred_lft forever 2: enp0s3: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc fq_codel state UP group default qlen 1000 link/ether 08:00:27:d3:c8:b1 brd ff:ff:ff:ff:ff:ff altname enx080027d3c8b1 inet 10.0.2.15/24 brd 10.0.2.255 scope global dynamic noprefixroute enp0s3 valid_lft 82077sec preferred_lft 71277sec inet6 fd17:625c:f037:2:a00:27ff:fed3:c8b1/64 scope global dynamic mngtmpaddr proto kernel_ra valid_lft 86020sec preferred_lft 14020sec inet6 fd17:625c:f037:2:7267:195d:80d5:5070/64 scope global dynamic mngtmpaddr noprefixroute valid_lft 86020sec preferred_lft 14020sec inet6 fe80::d39e:c03b:56a0:f31/64 scope link valid_lft forever preferred_lft forever 3: enp0s8: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc fq_codel state UP group default qlen 1000 link/ether 08:00:27:34:f3:fb brd ff:ff:ff:ff:ff:ff altname enx08002734f3fb inet 192.168.100.10/24 brd 192.168.100.255 scope global enp0s8 valid_lft forever preferred_lft forever inet6 fe80::a00:27ff:fe34:f3fb/64 scope link proto kernel_ll valid_lft forever preferred_lft forever 4: wg0: <POINTOPOINT,NOARP,UP,LOWER_UP> mtu 1420 qdisc noqueue state UNKNOWN group default qlen 1000 link/none inet 10.0.0.2/24 scope global wg0 valid_lft forever preferred_lft forever ibai@vbox:~$ ping -c3 10.0.0.1 PING 10.0.0.1 (10.0.0.1) 56(84) bytes of data. ^C --- 10.0.0.1 ping statistics --- 3 packets transmitted, 0 received, 100% packet loss, time 2030ms

Mostrar más

1:15

Las reglas están bien puestas y el DNAT tiene 1 paquete procesado (buena señal). El problema es que el paquete llega al router pero no vuelve, o WireGuard no está levantado en VM1. Vamos a diagnosticar:

## 1. Comprobar estado WireGuard en ambas VMs

bash

```bash
# En VM1
sudo wg show

# En VM2
sudo wg show
```

Dime qué devuelve cada una. Lo importante es ver si hay `latest handshake` y si el `listening port` aparece en VM1.

## 2. Tcpdump en el router para ver qué pasa con los paquetes

Abre **tres terminales** simultáneas en el router:

bash

```bash
# Terminal 1 — ver si llegan paquetes de VM2
sudo tcpdump -i enp0s9 udp port 51820 -n

# Terminal 2 — ver si se reenvían a VM1
sudo tcpdump -i enp0s8 udp port 51820 -n

# Terminal 3 — ver si VM1 responde
sudo tcpdump -i enp0s8 -n
```

Luego desde VM2:

bash

```bash
ping -c3 10.0.0.1
```

Y dime en cuál de los tres tcpdump aparecen paquetes. Eso nos dice exactamente dónde se corta el flujo.

vm1

interface: wg0 public key: quViPiqInLTZEUYnJ8X20xe8b5KLPqe0lADnQ56AHiE= private key: (hidden) listening port: 51820 peer: m1jEjOhNV76U/MBiifRIsAraNGVhuNozbh3lfxaGOmU= allowed ips: 10.0.0.2/32

vm2 interface: wg0 public key: m1jEjOhNV76U/MBiifRIsAraNGVhuNozbh3lfxaGOmU= private key: (hidden) listening port: 48590 peer: quViPiqInLTZEUYnJ8X20xe8b5KLPqe0lADnQ56AHiE= endpoint: 192.168.100.1:51820 allowed ips: 10.0.0.1/32 transfer: 0 B received, 7.80 KiB sent persistent keepalive: every 25 seconds

ibai@vbox:~$ sudo tcpdump -i enp0s9 udp port 51820 -n tcpdump: verbose output suppressed, use -v[v]... for full protocol decode listening on enp0s9, link-type EN10MB (Ethernet), snapshot length 262144 bytes 01:16:47.514975 IP 192.168.100.10.48590 > 192.168.100.1.51820: UDP, length 148 01:16:52.635368 IP 192.168.100.10.48590 > 192.168.100.1.51820: UDP, length 148 01:16:58.010991 IP 192.168.100.10.48590 > 192.168.100.1.51820: UDP, length 148 01:17:03.131441 IP 192.168.100.10.48590 > 192.168.100.1.51820: UDP, length 148 01:17:08.252560 IP 192.168.100.10.48590 > 192.168.100.1.51820: UDP, length 148 01:17:13.371614 IP 192.168.100.10.48590 > 192.168.100.1.51820: UDP, length 148 01:17:18.747468 IP 192.168.100.10.48590 > 192.168.100.1.51820: UDP, length 148

ibai@vbox:~$ sudo tcpdump -i enp0s8 udp port 51820 -n [sudo] password for ibai: tcpdump: verbose output suppressed, use -v[v]... for full protocol decode listening on enp0s8, link-type EN10MB (Ethernet), snapshot length 262144 bytes 01:16:52.635392 IP 192.168.100.10.48590 > 192.168.100.10.51820: UDP, length 148 01:16:58.011013 IP 192.168.100.10.48590 > 192.168.100.10.51820: UDP, length 148 01:17:03.131471 IP 192.168.100.10.48590 > 192.168.100.10.51820: UDP, length 148 01:17:08.252609 IP 192.168.100.10.48590 > 192.168.100.10.51820: UDP, length 148 01:17:13.371649 IP 192.168.100.10.48590 > 192.168.100.10.51820: UDP, length 148 01:17:18.747512 IP 192.168.100.10.48590 > 192.168.100.10.51820: UDP, length 148 01:17:23.868092 IP 192.168.100.10.48590 > 192.168.100.10.51820: UDP, length 148

ibai@vbox:~$ sudo tcpdump -i enp0s8 -n [sudo] password for ibai: tcpdump: verbose output suppressed, use -v[v]... for full protocol decode listening on enp0s8, link-type EN10MB (Ethernet), snapshot length 262144 bytes 01:16:57.806434 ARP, Request who-has 192.168.100.10 tell 192.168.100.1, length 28 01:16:57.808471 ARP, Reply 192.168.100.10 is-at 08:00:27:d5:23:ef, length 46 01:16:58.011012 IP 192.168.100.10.48590 > 192.168.100.10.51820: UDP, length 148 01:17:03.131469 IP 192.168.100.10.48590 > 192.168.100.10.51820: UDP, length 148 01:17:08.252606 IP 192.168.100.10.48590 > 192.168.100.10.51820: UDP, length 148 01:17:13.371647 IP 192.168.100.10.48590 > 192.168.100.10.51820: UDP, length 148 01:17:18.747510 IP 192.168.100.10.48590 > 192.168.100.10.51820: UDP, length 148 01:17:23.868090 IP 192.168.100.10.48590 > 192.168.100.10.51820: UDP, length 148 01:17:29.243626 IP 192.168.100.10.48590 > 192.168.100.10.51820: UDP, length 148

Mostrar más

1:17

Ves exactamente el problema en el tcpdump de enp0s8:

```
192.168.100.10.48590 > 192.168.100.10.51820
```

**El DNAT funciona** (el destino cambió de `192.168.100.1` a `192.168.100.10`), pero **el MASQUERADE no está funcionando** — el origen sigue siendo `192.168.100.10` (VM2). VM1 recibe un paquete que parece venir de sí misma y lo descarta.

El problema es que el MASQUERADE se aplica en POSTROUTING pero el paquete, después del DNAT, el kernel lo enruta hacia `192.168.100.10` que está en la misma interfaz `enp0s8` — y en ese caso el MASQUERADE no se dispara correctamente.

Necesitas forzar el SNAT con una IP concreta en vez de MASQUERADE:

bash

```bash
# Eliminar el MASQUERADE actual
sudo iptables -t nat -D POSTROUTING -o enp0s8 -j MASQUERADE

# Añadir SNAT explícito: cuando el paquete va a VM1, el origen se convierte en la IP del router en enp0s8
sudo iptables -t nat -A POSTROUTING -o enp0s8 -p udp --dport 51820 \
  -j SNAT --to-source 192.168.100.1
```

Verifica:

bash

```bash
sudo iptables -t nat -L -n -v --line-numbers
```

Debería quedar así en POSTROUTING:

```
SNAT  udp  --  *  enp0s8  0.0.0.0/0  0.0.0.0/0  udp dpt:51820  to:192.168.100.1
```

Ahora el flujo será:

```
VM2: 192.168.100.10:48590 → 192.168.100.1:51820
  DNAT  → destino cambia a 192.168.100.10:51820
  SNAT  → origen cambia a 192.168.100.1
VM1 ve: 192.168.100.1 → 192.168.100.10:51820  ✓
```

Prueba el ping desde VM2 y dime qué muestra `sudo wg show` en VM1.

ibai@vbox:~$ sudo iptables -t nat -D POSTROUTING -o enp0s8 -j MASQUERADE ibai@vbox:~$ sudo iptables -t nat -A POSTROUTING -o enp0s8 -p udp --dport 51820 \ -j SNAT --to-source 192.168.100.1 ibai@vbox:~$ sudo iptables -t nat -L -n -v --line-numbers Chain PREROUTING (policy ACCEPT 2 packets, 88 bytes) num pkts bytes target prot opt in out source destination 1 1 176 DNAT udp -- enp0s9 * 0.0.0.0/0 0.0.0.0/0 udp dpt:51820 to:192.168.100.10:51820 Chain INPUT (policy ACCEPT 0 packets, 0 bytes) num pkts bytes target prot opt in out source destination Chain OUTPUT (policy ACCEPT 0 packets, 0 bytes) num pkts bytes target prot opt in out source destination Chain POSTROUTING (policy ACCEPT 3 packets, 201 bytes) num pkts bytes target prot opt in out source destination 1 0 0 SNAT udp -- * enp0s8 0.0.0.0/0 0.0.0.0/0 udp dpt:51820 to:192.168.100.1

Mostrar más

1:18

Bien, las reglas están. Ahora prueba desde VM2:

bash

```bash
ping -c3 10.0.0.1
```

Y simultáneamente en el router comprueba si ahora el origen cambia:

bash

```bash
sudo tcpdump -i enp0s8 udp port 51820 -n
```

Ahora deberías ver `192.168.100.1.XXXX > 192.168.100.10.51820` en vez de `192.168.100.10 > 192.168.100.10`. Y en VM1:

bash

```bash
sudo wg show
```

¿Aparece `latest handshake`?

ibai@vbox:~$ sudo tcpdump -i enp0s8 udp port 51820 -n tcpdump: verbose output suppressed, use -v[v]... for full protocol decode listening on enp0s8, link-type EN10MB (Ethernet), snapshot length 262144 bytes 01:19:04.219383 IP 192.168.100.10.48590 > 192.168.100.10.51820: UDP, length 148 01:19:09.338995 IP 192.168.100.10.48590 > 192.168.100.10.51820: UDP, length 148 01:19:14.460280 IP 192.168.100.10.48590 > 192.168.100.10.51820: UDP, length 148 01:19:19.835517 IP 192.168.100.10.48590 > 192.168.100.10.51820: UDP, length 148 01:19:25.212602 IP 192.168.100.10.48590 > 192.168.100.10.51820: UDP, length 148 01:19:30.587284 IP 192.168.100.10.48590 > 192.168.100.10.51820: UDP, length 148

1:19

El SNAT no se está aplicando todavía. El origen sigue siendo `192.168.100.10`. Fíjate que el contador de la regla SNAT está a `0 packets` — no está matchando ningún paquete.

El problema es el orden: el DNAT cambia el destino a `192.168.100.10:51820`, pero el SNAT tiene condición `--dport 51820` y en ese punto del pipeline (POSTROUTING) el puerto destino ya fue reescrito por el DNAT... o el paquete no pasa por POSTROUTING porque el kernel lo considera local.

Simplifica la regla SNAT — quita la condición de puerto:

bash

```bash
# Eliminar la regla actual
sudo iptables -t nat -D POSTROUTING -o enp0s8 -p udp --dport 51820 \
  -j SNAT --to-source 192.168.100.1

# Regla más amplia — cualquier tráfico que salga por enp0s8 hacia VM1
sudo iptables -t nat -A POSTROUTING -o enp0s8 \
  -s 192.168.100.0/24 \
  -j SNAT --to-source 192.168.100.1
```

Verifica contadores:

bash

```bash
sudo iptables -t nat -L -n -v --line-numbers
```

Y vuelve a mirar el tcpdump. Si sigue sin cambiar el origen, el problema es que el kernel está enrutando el paquete de forma distinta — dime y miramos con `conntrack`:

bash

```bash
sudo apt install -y conntrack
sudo conntrack -E -p udp --dport 51820
```

Eso nos muestra en tiempo real cómo el kernel está traduciendo las conexiones.

  

[Claude es IA y puede cometer errores. Por fav](https://support.anthropic.com/en/articles/8525154-claude-is-providing-incorrect-or-misleading-responses-what-s-going-on)
```