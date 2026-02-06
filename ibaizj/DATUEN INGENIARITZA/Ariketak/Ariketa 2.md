
![[5 Datuen Ingeniaritza - 2.2.3 Datu masiboentzako azpiegitura oinarriak - Oinarrizko kontzeptuak - Ariketa 2.pdf]]

# 1. Opentofu
## 1.1.  Instalatu OpenTofu
[[OpenTofu#Install OpenTofu|Ikusi instalazioaren pausoak]]

## 1.2. Proxmox kudeatu dezan konfiguratu

```
https://search.opentofu.org/provider/telmate/proxmox/latest
```

- Sortu direktorio bat
```
mkdir -p ~/OpenTofu/proxmox-lxc
cd ~/OpenTofu/proxmox-lxc
```

- Terraform artxibo bat sortu
```
touch main.tf
```

- Terraform provider-a konfiguratu

```
terraform {
  required_providers {
    proxmox = {
      source = "telmate/proxmox"
      version = "3.0.2-rc07"
    }
  }
}
```









