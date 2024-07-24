# Terraform set 3 or More VM to Automatic Setup Requirement YAVA with Ansible Playbook

## Environment
- KVM
- Terraform v1.2.6
- Centos 7 / Fedora
- Config terraform 3 or more VM (im already include sample 3 vm config in this book)

## Installation
1. First of all install ansible dependencies, if you havent install env, install it with these command below
if command not found try using `python3 -m venv venv`
```
python -m venv venv
```
update venv with source
```
source ./venv/bin/activate
```
after that upgrade pip command
```
pip install --upgrade pip
```

2. Then, you need to install Ansible, to install Ansible run this command below:
```
$ virtualenv -p python3 .venv
$ source .venv/bin/activate
$ pip install ansible
```

3. Prepare config terraform 3 or more VM, here's the example config 3 VM, named *centos7.tf*
```
variable "vm_name" {
  type = list(string)
  default = ["vm1","vm2","vm3"]
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

variable "ssh_username" {
  description = "the ssh user to use"
  default     = "bintang"
}

variable "ssh_private_key" {
  description = "the private key to use"
  default     = "~/.ssh/id_rsa"
}

provider "libvirt" {
  uri = "qemu+ssh://bintang@192.168.10.110/system"
}


resource "libvirt_volume" "volume-vm1" {
  name = "${var.vm_name[0]}.img"
  pool = "default"
  source = "/home/bintang/Downloads/CentOS-7-x86_64-GenericCloud.qcow2"
  format = "qcow2"
}

resource "libvirt_volume" "volume-vm2" {
  name = "${var.vm_name[1]}.img"
  pool = "default"
  source = "/home/bintang/Downloads/CentOS-7-x86_64-GenericCloud.qcow2"
  format = "qcow2"
}

resource "libvirt_volume" "volume-vm3" {
  name = "${var.vm_name[2]}.img"
  pool = "default"
  source = "/home/bintang/Downloads/CentOS-7-x86_64-GenericCloud.qcow2"
  format = "qcow2"
}


data "template_file" "user_data" {
  template = file("${path.module}/cloud_init.cfg")
   vars = {
    hostname = "var.vm_name"
    domain = "var.domain"
  }
}
data "template_file" "network_config" {
  template = file("${path.module}/network_config.yml")
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
  qemu_agent = true

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

  provisioner "remote-exec" {
    inline = [
      "echo 'Hello World'"
    ]
      
    connection {
      type              = "ssh"
      user              = var.ssh_username
      host              = libvirt_domain.domain_vm1.network_interface.0.addresses.0
      private_key       = file(var.ssh_private_key)
      timeout           = "2m"
    }
  }
  provisioner "local-exec" {
    command = <<EOT
      echo "[centos]" > centos.ini
      echo "${libvirt_domain.domain_vm1.network_interface.0.addresses.0}" >> centos.ini
      echo "[centos:vars]" >> centos.ini
      echo "ansible_ssh_common_args='-o StrictHostKeyChecking=no -o UserKnownHostsFile=/dev/null -o ProxyCommand=\"ssh -W %h:%p -q ams-kvm-remote-host\"'" >> centos.ini
      ansible-playbook -u ${var.ssh_username} --private-key ${var.ssh_private_key} -i centos.ini playbook.yml
      EOT
  }
}

#}

# Define KVM domain to create
resource "libvirt_domain" "domain_vm2" {
  name   = var.vm_name[1]
  memory = var.memory
  vcpu   = var.cpu
  qemu_agent = true
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
  provisioner "remote-exec" {
    inline = [
      "echo 'Hello World'"
    ]
      
    connection {
      type              = "ssh"
      user              = var.ssh_username
      host              = libvirt_domain.domain_vm2.network_interface.0.addresses.0
      private_key       = file(var.ssh_private_key)
      timeout           = "2m"
    }
  }
  provisioner "local-exec" {
    command = <<EOT
      echo "[centos]" > centos.ini
      echo "${libvirt_domain.domain_vm2.network_interface.0.addresses.0}" >> centos.ini
      echo "[centos:vars]" >> centos.ini
      echo "ansible_ssh_common_args='-o StrictHostKeyChecking=no -o UserKnownHostsFile=/dev/null -o ProxyCommand=\"ssh -W %h:%p -q ams-kvm-remote-host\"'" >> centos.ini
      ansible-playbook -u ${var.ssh_username} --private-key ${var.ssh_private_key} -i centos.ini playbook.yml
      EOT
  }
}



# Define KVM domain to create
resource "libvirt_domain" "domain_vm3" {
  name   = var.vm_name[2]
  memory = var.memory
  vcpu   = var.cpu
  qemu_agent = true
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
  provisioner "remote-exec" {
    inline = [
      "echo 'Hello World'"
    ]
      
    connection {
      type              = "ssh"
      user              = var.ssh_username
      host              = libvirt_domain.domain_vm3.network_interface.0.addresses.0
      private_key       = file(var.ssh_private_key)
      timeout           = "2m"
    }
  }
  provisioner "local-exec" {
    command = <<EOT
      echo "[centos]" > centos.ini
      echo "${libvirt_domain.domain_vm3.network_interface.0.addresses.0}" >> centos.ini
      echo "[centos:vars]" >> centos.ini
      echo "ansible_ssh_common_args='-o StrictHostKeyChecking=no -o UserKnownHostsFile=/dev/null -o ProxyCommand=\"ssh -W %h:%p -q ams-kvm-remote-host\"'" >> centos.ini
      ansible-playbook -u ${var.ssh_username} --private-key ${var.ssh_private_key} -i centos.ini playbook.yml
      EOT
   }
}
#    connection {
#      type              = "ssh"
#      user              = var.ssh_username
#      host              = libvirt_domain.domain_vm3.network_interface.0.addresses.0
#      private_key       = file(var.ssh_private_key)
#      timeout           = "2m"
#    }


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
just double check where you need to change values like name, cpu, memory, and etc to suite your environment

4. Create file named *main.tf* and add these config
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
Also don't forget to change value like example the version if you're using different terraform version

5. Create file again named *cloud_init.cfg*
Change value root password if you want the different password as you like, and change value user name, ssh-rsa key, but dont forget to change user name same as value in the file *centos7.tf*, otherwise it will error because some value aren't the same
```
#cloud-config
# vim: syntax=yaml
#
# ***********************
#       ---- for more examples look at: ------
# ---> https://cloudinit.readthedocs.io/en/latest/topics/examples.html
# ******************************
#
# This is the configuration syntax that the write_files module
# will know how to understand. encoding can be given b64 or gzip or (gz+b64).
# The content will be decoded accordingly and then written to the path that is
# provided.
#
# Note: Content strings here are truncated for example purposes.
bootcmd:
  - echo 192.168.0.1 gw.homedns.xyz >> /etc/hosts
runcmd:
 - [ ls, -l, / ]
 - [ sh, -xc, "echo $(date) ': hello world!'" ]
ssh_pwauth: true
disable_root: false
chpasswd:
  list: |
     root:password
  expire: false
users:
  - name: bintang
# gecos: "bintang"
#    primary_group: bintang
    sudo: ALL=(ALL) NOPASSWD:ALL
    groups: users, admin
    home: /home/bintang
    shell: /bin/bash
    lock_passwd: false
    ssh_authorized_keys: ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABgQDIwj89b5iI1VAhsFx6KPGTZxrgS+lxwwK+62kgCwmBcOj9zOZud0rnNuhs7ZSF/V4bB+CbLGeHQ0jadN40QG0xBNLX1afab7wkXSYQuDKo6ubOO/wt3glE1PcVnYLhJqXMHbqXIeD4+02UZdXSOSTmEzcP3uGzQX2mn8Z4j24FFFAOVllWJnXmTLMC8giemyqh12bEwcZGquseLSc4pbtgOp3lDSb0thg8MiUfnbH/Hacq64TbiBBfU4RgHBmvTrtgDhrSXD25/69IVU5EZFAQNoApgJXt/G/16QI3PXIgetYYI1u8jLxndA215GakUT5cyw9khiJkB6ae+RHJ+XDbohlqKxo/c5AnQ8NTyEdkqqnypjUrzY4eOSmSc8AdQTDc1IgxLMKfulBImRYUKQgm8xn29XyOavXR9YzvE83/LfUoXuiZjwHGV+gz6CZHm4KrRnpbwsGCYfH4JKwsDUh1iT+XE6LQEEIfM2RWp0ffv1kOtAZdMwn+lVt+/mq35is= bintang@fedora
hostname: "${hostname}.${domain}"
```

6. 
