
```
terraform {
  required_providers {
    proxmox = {
      source  = "telmate/proxmox"
      version = "~> 3.0.2-rc07"
    }
    null = {
      source  = "hashicorp/null"
      version = "~> 3.0"
    }
    google = {
      source = "opentofu/google"
      version = "~> 7.22.0"
    }
  }
}

provider "proxmox" {
  pm_api_url = var.pm_api_url
  pm_user = var.pm_user
  pm_password = var.pm_password
  pm_tls_insecure = true
}

provider "google" {
  credentials = file("~/gcp/ibai-zorrilla-489210-093cdafbf90b.json")
  project = "ibai-zorrilla-489210"
  region = "europe-west1"
}

```

![[Pasted image 20260309111253.png]]

![[Pasted image 20260309111208.png]]

![[Pasted image 20260309112224.png]]


Instalatu GCP CLI
```
curl https://sdk.cloud.google.com | bash
exec -l $SHELL
```

login
```
gcloud auth application-default login
```

