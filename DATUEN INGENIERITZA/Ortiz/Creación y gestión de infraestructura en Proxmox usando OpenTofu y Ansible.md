
Tofu
```shell
apt install curl
curl --proto '=https' --tlsv1.2 -fsSL https://get.opentofu.org/install-opentofu.sh -o install-opentofu.sh
chmod +x install-opentofu.sh
./install-opentofu.sh --install-method deb
rm -f install-opentofu.sh
```

Ansible
```bash
apt install -y ansible
ansible --version
```

Creamos el proyecto de infrastructura:
```bash
mkdir infra
cd infra
```

Creamos los archivos necesarios:
```bash
touch providers.tf main.tf variables.tf
```

---

Asignamos permisios al token:
```
Datacenter
 └── Permissions
     └── Add
         └── API Token Permission
```

- **Path `/`** → permisos globales
- **API Token `root@pam!opentofu`**
- **Role `PVEAdmin` / `Administrator`**
- **Propagate = true**

Configurar opentofu para proxmox:
```bash
nano providers.tf
```

Configuramos `provider.tf`:

```shell
terraform {
  required_providers {
    proxmox = {
      source  = "telmate/proxmox"
      version = "2.8.0"
    }
  }
}

provider "proxmox" {
  pm_api_url      = "https://192.168.8.1:8006/api2/json"
  pm_user         = "root@pam"
  pm_password     = "rootroot"
  pm_tls_insecure = true
}
```

Para borrar el cache de tofu:
```
cd ~/infra
rm -rf .terraform
rm -f .terraform.lock.hcl
```

Inicializamos tofu:
```bash
tofu init
```

Ahora configuramos el `main.tf`

```txt
NOMBRE_DEL_NODO = proxmox
ostemplate = debian-13-standard_13.1-2_amd64.tar.zst
storage = local-lvm
```

Como mirar el `ostemplate`:

1. Web de proxmox
2. ```
   Datacenter
  └── TU_NODO
      └── local
          └── CT Templates
   ```
3.  Instalamos la template de `debian-12-standard`
4. Al descargar saldrá un nombre:
```
debian-12-standard_12.12-1_amd64.tar.zst
```

Como mirar el `storage`:
1. ```
   Datacenter
 └── TU_NODO   (por ejemplo: proxmox)
     └── Storage
   ```
2. Usamos `local.lvm`

Configurado `main.tf`
```shell
resource "proxmox_lxc" "lxc_debian" {
  target_node = var.proxmox_node
  hostname    = "lxc-opentofu"
  vmid        = 101

  ostemplate = "local:vztmpl/debian-12-standard_12.12-1_amd64.tar.zst"

  cores  = 1
  memory = 1024
  swap   = 512

  rootfs {
    storage = "local-lvm"
    size    = "8G"
  }

network {
  name   = "eth0"
  bridge = "vmbr1"
  ip     = "192.168.12.50/23"
  gw     = "192.168.13.254"
}

features {
  nesting = true
}

  password = "root123"
  unprivileged = false
  start        = true
}
```

Configuramos `variables.tf`:
```shell
variable "proxmox_node" {
  default = "pve1"
}
```

Ejecutamos:
```
tofu plan
```

Salida:
```bash
Plan: 1 to add, 0 to change, 0 to destroy.
```

Por último, ejecutamos:
```shell
tofu apply
```

```shell
Do you want to perform these actions?
```

Value:  `yes`

---

Importamos el contenedor de OpenTofu:
```shell
tofu import proxmox_lxc.lxc_debian proxmox/lxc/101
```

Comprobamos:
```shell
tofu state list
```

--- 

Pare eliminar:
```
pct stop 101
pct destroy 101
```

---

## Preparar el LXC para Ansible

Miramos la IP del LXC:
```shell
pct exec 101 -- ip a
```

Si vemos que a la interfaz no le ha dado ip por DHCP

Entramos en el contenedor:
```shell
pct exec 101 -- bash
```
```shell
ip a
```
Levantamos la interfaz:
```shell
ip link set eth0 up
```

Si vemos que no tiene IP, es porque el DHCP no se lo da, por lo tanto:

Comprobamos si hay `dhclient` dentro del LXC:
```shell
which dhclient
```

Si no sale nada, instálalo:
```shell
apt update
apt install -y isc-dhcp-client
```

Pedimos la IP por DHCP:
```shell
dhclient eth0
```

No va a dar una IP porque no hay un servidor DHCP, por lo tanto, asignamos una estática.

```shell
nano /etc/network/interfaces
```

```
auto lo
iface lo inet loopback

auto eth0
iface eth0 inet static
  address 192.168.100.50
  netmask 255.255.255.0
  gateway 192.168.100.10
```

Para que desde debian haga ping a LXC, añadimos una ruta por defecto:
```
ip route add default via 192.168.100.10 dev eth0
```

---

## Crear un playbook para actualizar proxmox y GCP

```bash
mkdir ansible
cd ansible
touch inventory.ini playbook.yml
```

Configuramos `inventory.ini`

```bash
nano inventory.ini
```
```ini
[proxmox]
192.168.8.1 ansible_user=root
ansible_python_interpreter=/usr/bin/python3

[gcp]
34.XX.XX.XX ansible_user=debian
```

Configuramos `update_proxmox.yml`

```bash
nano update_proxmox.yml
```
```yaml
- name: Actualizar Proxmox
  hosts: proxmox
  become: true

  tasks:
    - name: Actualizar cache APT
      apt:
        update_cache: yes

    - name: Actualizar paquetes
      apt:
        upgrade: dist
```

Configuramos `update_gpc.yml`

```yml
- name: Actualizar máquinas GCP
  hosts: gcp
  become: true

  tasks:
    - name: Actualizar cache APT
      apt:
        update_cache: yes

    - name: Actualizar paquetes
      apt:
        upgrade: dist

```

Ejecutar playbook:
```shell
ansible-playbook -i inventory.ini update_proxmox.yml
```
```
ansible-playbook -i inventory.ini update_gcp.yml
```


IMPORTANTE

Hay que quitar la contraseña que pide cuando te conecta a proxmox desde debian via SSH.
Lo que vamos a decirle a proxmox es que Debian es de confianza y que pueda entrar sin pedir contraseña

1. Comprobamos si Debian tiene clave:
```bash
ls ~/.ssh
```

Si ves algo como:
```rust
id_rsa
id_rsa.pub
```
Ir al paso 2.

Si no hay que ejecutar:
```shell
ssh-keygen
```
Enter a todo sin escribir nada.

2. Copiar la clave a proxmox
```shell
ssh-copy-id root@192.168.8.1
```

3. Comprobamos que funciona:
```shell
ssh root@192.168.100.10
```


---


```
export TF_LOG=DEBU
```