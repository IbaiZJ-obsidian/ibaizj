
# Sortu Debian makina birtuala

```
# VM-a sortu

VBoxManage createvm --name "debian13" --groups "/DatuIngenieritza" --ostype Debian_64 --register

VBoxManage modifyvm "debian13" --memory 2048 --vram 20 --cpus 2 --rtc-use-utc on --graphicscontroller vmsvga --firmware bios

VBoxManage modifyvm "debian13" --nic1 nat --nic2 intnet --intnet2 "hodeipribatu-kudeatzailea" --macaddress2 000102030405

# Sortu port-forwarding

VBoxManage modifyvm "debian13" --nat-pf1 '"debian13 ssh",tcp,,2205,,22'

# Sortu 2 puerto ISOa eta diskoa gehitzeko

VBoxManage storagectl "debian13" --name SATA --add sata --controller IntelAhci --portcount 2

# Gehitu Debian13-ko ISOa

VBoxManage storageattach "debian13" --storagectl SATA --device 0 --medium "C:\Users\iibai\iso\debian-13.3.0-amd64-netinst.iso" --port 0 --type dvddrive

# Sortu eta gehitu makinaren diskoa

VBoxManage createmedium disk --filename "C:\Users\iibai\VirtualBox VMs\DatuIngenieritza\debian13\debian13.vdi" --size 122880 --format VDI

VBoxManage storageattach "debian13" --storagectl SATA --device 0 --medium "C:\Users\iibai\VirtualBox VMs\DatuIngenieritza\debian13\debian13.vdi" --port 1 --type hdd

VBoxManage snapshot "debian13" take "Instalazio Aurretik"
```

# Debian Istalakuntza

- 2 network interfaze badauz, nat sarearen interfazea aukeratu printzipal bezala
![[Pasted image 20260212105457.png]]

## Interfaseen konfigurazioa

```
# konfigurazio fitxategia:
nano /etc/network/interfaces
```

- Dagoen konfigurazioa
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
```