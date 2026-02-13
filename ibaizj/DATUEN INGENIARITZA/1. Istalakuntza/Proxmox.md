
# Sortu Proxmox makina birtuala

```
# VM-a sortu

VBoxManage createvm --name "proxmox" --groups "/DatuIngenieritza" --ostype Debian_64 --register

VBoxManage modifyvm "proxmox" --memory 4096 --vram 20 --cpus 4 --rtc-use-utc on --graphicscontroller vmsvga --firmware bios

VBoxManage modifyvm "proxmox" --nic1 nat --nic2 intnet --intnet2 "hodeipribatu-gestioa" --macaddress2 000102030406 --nic3 intnet --intnet3 "hodeipribatu-datuak" --macaddress3 000102030407

# Sortu port-forwarding

VBoxManage modifyvm "proxmox" --nat-pf1 '"proxmox ssh",tcp,,2206,,22'

# Sortu 2 puerto ISOa eta diskoa gehitzeko

VBoxManage storagectl "proxmox" --name SATA --add sata --controller IntelAhci --portcount 2

# Gehitu Proxmox-ko ISOa

VBoxManage storageattach "proxmox" --storagectl SATA --device 0 --medium "C:\Users\iibai\iso\proxmox-ve_9.1-1.iso" --port 0 --type dvddrive

# Sortu eta gehitu makinaren diskoa

VBoxManage createmedium disk --filename "C:\Users\iibai\VirtualBox VMs\DatuIngenieritza\proxmox\proxmox.vdi" --size 122880 --format VDI

VBoxManage storageattach "proxmox" --storagectl SATA --device 0 --medium "C:\Users\iibai\VirtualBox VMs\DatuIngenieritza\proxmox\proxmox.vdi" --port 1 --type hdd

VBoxManage snapshot "proxmox" take "Instalazio Aurretik"
```

```
# Istalazio ostean .iso-a kendu !!!
VBoxManage storageattach "proxmox" --storagectl SATA --port 0 --device 0 --type dvddrive --medium none
```