# Terraform with KVM CentOS 7

## Environment
- KVM
- Terraform
- Centos 7 / Fedora

## Install KVM

1. First of all, install epel-release with this command
```
sudo yum -y install epel-release
```

2. After that install libvirt and other dependency
```
sudo yum -y install gcc libvirt libvirt-devel qemu-kvm virt-install virt-top libguestfs-tools bridge-utils
```
Confirm that the kernel module are loaded with
```
sudo lsmod | grep kvm
```

Start and enable libvirtd service
```
sudo systemctl start libvirtd && sudo systemctl enable libvirtd
```

## Install Terraform on Linux

1. Install the required tools with command:
```
sudo yum install curl wget unzip
```

2. Install Terraform, with command
```
wget https://releases.hashicorp.com/terraform/0.13.3/terraform_0.13.3_linux_amd64.zip
```

```
unzip terraform_0.13.3_linux_amd64.zip
```

```
sudo mv terraform /usr/local/bin/terraform
```

3. Install Ansible
```
virtualenv -p python3 .venv
```

```
source .venv/bin/activate
```

```
pip install ansible
```

4. then make tool accessible to all user account
```
which terraform
```
Check the version installed
```
terraform --version
```

## Config Terraform

1. Create workspace directory for demonstration
```
mkdir -p ~/workspace/terraform-kvm-example/
cd ~/workspace/terraform-kvm-example/
```

2. change directory to the workspace and create file named *main.tf*
```
terraform {
  required_providers {
    libvirt = {
      source = "dmacvicar/libvirt"
      version = "0.6.2"
    }
  }
}
```

3. Then *centos7.tf*, you need to change some values that suit your environment:

```
variable "vm_name" {
  type = list(string)
  default = ["node-01","node-02","node-03"]
}

variable "domain" {
  type = string
  default = "example.com"
}

variable "memory" {
  type = string
  default = "512"
}

variable "cpu" {
  type = number
  default = 1
}

provider "libvirt" {
  uri = "qemu:///system"
}


resource "libvirt_volume" "volume-vm1" {
  name = "${var.vm_name[0]}.img"
  pool = "default"
  source = "/home/fajar/Downloads/CentOS-7-x86_64-GenericCloud.qcow2"
  format = "qcow2"
}

resource "libvirt_volume" "volume-vm2" {
  name = "${var.vm_name[1]}.img"
  pool = "default"
  source = "/home/fajar/Downloads/CentOS-7-x86_64-GenericCloud.qcow2"
  format = "qcow2"
}

resource "libvirt_volume" "volume-vm3" {
  name = "${var.vm_name[2]}.img"
  pool = "default"
  source = "/home/fajar/Downloads/CentOS-7-x86_64-GenericCloud.qcow2"
  format = "qcow2"
}


data "template_file" "user_data" {
  template = file("${path.module}/cloud_init.cfg")

  vars = {
   hostname = "var.vm_name"
   domain = "var.domain"
  }
}

resource "libvirt_cloudinit_disk" "commoninit" {
  name           = "commoninit-vm1.iso"
  user_data      = data.template_file.user_data.rendered
  pool           = "default"
}

resource "libvirt_cloudinit_disk" "commoninit2" {
  name           = "commoninit-vm2.iso"
  user_data      = data.template_file.user_data.rendered
  pool           = "default"
}

resource "libvirt_cloudinit_disk" "commoninit3" {
  name           = "commoninit-vm3.iso"
  user_data      = data.template_file.user_data.rendered
  pool           = "default"
}


# Define KVM domain to create
resource "libvirt_domain" "domain_vm1" {
  name   = var.vm_name[0]
  memory = var.memory
  vcpu   = var.cpu

  cloudinit = libvirt_cloudinit_disk.commoninit.id

  disk {
    volume_id = libvirt_volume.volume-vm1.id

  }
  console {
    type = "pty"
    target_type = "serial"
    target_port = "0"
  }

  graphics {
    type = "spice"
    listen_type = "address"
    autoport = true
  }

  network_interface {
    network_name = "default"
    wait_for_lease = true
  }
}

# Define KVM domain to create
resource "libvirt_domain" "domain_vm2" {
  name   = var.vm_name[1]
  memory = var.memory
  vcpu   = var.cpu

  cloudinit = libvirt_cloudinit_disk.commoninit.id

  disk {
    volume_id = libvirt_volume.volume-vm2.id

  }
  
  console {
    type = "pty"
    target_type = "serial"
    target_port = "0"
  }

  graphics {
    type = "spice"
    listen_type = "address"
    autoport = true
  }

  network_interface {
    network_name = "default"
    wait_for_lease = true
  }
}

# Define KVM domain to create
resource "libvirt_domain" "domain_vm3" {
  name   = var.vm_name[2]
  memory = var.memory
  vcpu   = var.cpu

  cloudinit = libvirt_cloudinit_disk.commoninit.id

  disk {
    volume_id = libvirt_volume.volume-vm3.id

  }

  console {
    type = "pty"
    target_type = "serial"
    target_port = "0"
  }

  graphics {
    type = "spice"
    listen_type = "address"
    autoport = true
  }

  network_interface {
    network_name = "default"
    wait_for_lease = true
  }
}


output "ip1" {
  value = libvirt_domain.domain_vm1.network_interface.0.addresses.0
}

output "ip2" {
  value = libvirt_domain.domain_vm2.network_interface.0.addresses.0
}

output "ip3" {
  value = libvirt_domain.domain_vm3.network_interface.0.addresses.0
}

```

4. create cloud_init.cfg
```
#cloud-config
# vim: syntax=yaml
#
# ***********************
# 	---- for more examples look at: ------
# ---> https://cloudinit.readthedocs.io/en/latest/topics/examples.html
# ******************************
#
# This is the configuration syntax that the write_files module
# will know how to understand. encoding can be given b64 or gzip or (gz+b64).
# The content will be decoded accordingly and then written to the path that is
# provided.
#
# Note: Content strings here are truncated for example purposes.
ssh_pwauth: True
disable_root: false
chpasswd:
  list: |
     root:password
  expire: false
users:
  - name: fajar
# gecos: "fajar"
#    primary_group: fajar
    sudo: ALL=(ALL) NOPASSWD:ALL
    groups: users, admin
    ssh_authorized_keys: ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABgQDIwj89b5iI1VAhsFx6KPGTZxrgS+lxwwK+62kgCwmBcOj9zOZud0rnNuhs7ZSF/V4bB+CbLGeHQ0jadN40QG0xBNLX1afab7wkXSYQuDKo6ubOO/wt3glE1PcVnYLhJqXMHbqXIeD4+02UZdXSOSTmEzcP3uGzQX2mn8Z4j24FFFAOVllWJnXmTLMC8giemyqh12bEwcZGquseLSc4pbtgOp3lDSb0thg8MiUfnbH/Hacq64TbiBBfU4RgHBmvTrtgDhrSXD25/69IVU5EZFAQNoApgJXt/G/16QI3PXIgetYYI1u8jLxndA215GakUT5cyw9khiJkB6ae+RHJ+XDbohlqKxo/c5AnQ8NTyEdkqqnypjUrzY4eOSmSc8AdQTDc1IgxLMKfulBImRYUKQgm8xn29XyOavXR9YzvE83/LfUoXuiZjwHGV+gz6CZHm4KrRnpbwsGCYfH4JKwsDUh1iT+XE6LQEEIfM2RWp0ffv1kOtAZdMwn+lVt+/mq35is= fajar@fedora
hostname: "${hostname}.${domain}"
packages:
  - wget
  - python3-pip
```

## Deploy Terraform

1. initialize terraform to download all the plugins

```
terraform init
```
```
Initializing the backend...

Initializing provider plugins...
- Reusing previous version of dmacvicar/libvirt from the dependency lock file
- Reusing previous version of hashicorp/template from the dependency lock file
- Using previously-installed hashicorp/template v2.2.0
- Using previously-installed dmacvicar/libvirt v0.6.2

```

```
terraform plan
```

```
terraform apply
```
and wait the process til it complete

## Test our VM
Hop into KVM Host and test out ssh connection

```
ssh root@your-ip-address
```

## Reference
- [https://computingforgeeks.com/how-to-install-terraform-on-linux/](https://computingforgeeks.com/how-to-install-terraform-on-linux/)
- [https://dev.to/ruanbekker/terraform-with-kvm-2d9e](https://dev.to/ruanbekker/terraform-with-kvm-2d9e)
- [https://computingforgeeks.com/how-to-provision-vms-on-kvm-with-terraform/](https://computingforgeeks.com/how-to-provision-vms-on-kvm-with-terraform/)
- [https://github.com/Mosibi/centos8-terraform](https://github.com/Mosibi/centos8-terraform)


