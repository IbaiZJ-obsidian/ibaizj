
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

- Gehitu proxmox provider-a
```
provider "proxmox" {
	
}
```

- Konfigurazio ezberdinak egin
```
# Konfiguratu endpoint-a
pm_api_url = "https://10.0.2.15:8006/api2/json"

# Usuarioa konfiguratu
pm_user = "root@pam"

# Pasahitza konfiguratu
pm_password = "Pasaitza123"

# Zertifikatu balidoa ez baduzu = true, bestela = false
# By default Proxmox Virtual Environment uses self-signed certificates.
pm_tls_insecure = true
```

- Sortu LXC kontenedorea
```
resource "proxmox_lxc" "mi_lxc" {
  hostname       = "lxc-test"
  ostemplate     = "local:vztmpl/debian-13-standard_13.0-1_amd64.tar.gz"
  storage        = "local-lvm"
  cores          = 2
  memory         = 2048
  rootfs         = "8G"
  swap           = 512
  password       = "Pasaitza123"

    network {
        name   = "eth0"
        bridge = "vmbr0"
        ip     = "10.0.2.15/24"
        gw     = "10.0.2.2"
    }
}
```

```
# Inicializa OpenTofu en este proyecto
tofu init

# Descarga los providers necesarios
tofu providers install
```

```
tofu plan
```

```
tofu apply
```



```
terraform {
    required_providers {
        proxmox = {
      source = "telmate/proxmox"
      version = "3.0.2-rc07"
    }
  }
}

provider "proxmox" {
    pm_api_url = "https://10.0.2.15:8006/api2/json"
    pm_user = "root@pam"
    pm_password = "Zoo22881"
    pm_tls_insecure = true
}

resource "proxmox_lxc" "mi_lxc" {
    hostname       = "lxc-test"
    ostemplate     = "local:vztmpl/debian-13-standard_13.0-1_amd64.tar.gz"
    storage        = "local-lvm"
    cores          = 2
    memory         = 2048
    rootfs         = "8G"
    swap           = 512
    password       = "Zoo228811"

    network {
        name   = "eth0"
        bridge = "vmbr0"
        ip     = "10.0.2.16/24"
        gw     = "10.0.2.2"
    }
}
```