
![[5 Datuen Ingeniaritza - 2.2.3 Datu masiboentzako azpiegitura oinarriak - Oinarrizko kontzeptuak - Ariketa 2.pdf#page=2]]

# 1. Opentofu
## 1.1.  Instalatu OpenTofu

```
https://opentofu.org/docs/intro/install/deb/
```

- Instalatu curl
```
apt install curl
```

```
curl --proto '=https' --tlsv1.2 -fsSL https://get.opentofu.org/install-opentofu.sh -o install-opentofu.sh
```

```
chmod +x install-opentofu.sh
```

```
./install-opentofu.sh --install-method deb
```

```
rm -f install-opentofu.sh
```


## 1.2. Proxmox kudeatu dezan konfiguratu

- Dokumentazio web-orria
```
https://search.opentofu.org/provider/telmate/proxmox/latest
```

- Sortu direktorio bat OpenTofuko proxmoxeko konfigurazioak egiteko
```
mkdir -p ~/OpenTofu/proxmox-lxc
cd ~/OpenTofu/proxmox-lxc
```

- OpenTofurako .tf artxibo bat sortu
```
touch main.tf
```

- Terraform provider-a konfiguratu main.tf-n

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
pm_api_url = "https://10.0.2.10:8006/api2/json"

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
resource "proxmox_lxc" "debian_container" {
    hostname = "debian-lxc"
    target_node = "proxmox"
    ostemplate = "local:vztmpl/debian-13-standard_13.1-2_amd64.tar.zst"

    cores = 2
    memory = 2048
    swap = 512

    rootfs {
        storage = "local-lvm"
        size = "8G"
    }

    network {
        name = "eth0"
        bridge = "vmbr0"
        ip = "dhcp"
    }

    password = "12345678"
    unprivileged = true
    onboot = true
    start = true
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

```
ls -lh /var/lib/vz/template/cache/
```


```
apt install ansible -y
```

```
ansible --version 
ansible-playbook --version
```