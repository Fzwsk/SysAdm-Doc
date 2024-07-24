# Terraform Error cant run command destroy

## Issue
```
│ Error: Invalid index
│ 
│   on centos7.tf line 169, in output "ip1":
│  169:   value = libvirt_domain.domain_vm1.network_interface.0.addresses.0
│     ├────────────────
│     │ libvirt_domain.domain_vm1.network_interface[0].addresses is empty list of string
│ 
│ The given key does not identify an element in this collection value: the collection has no elements.
```

## Environment
- Fedora 35
- Terraform v1.2.6

## Resolution
1. menghapus manual state yang menyebabkan tidak bisa running, dengan cara running command
```
terraform state list
```

```
data.template_file.user_data
libvirt_cloudinit_disk.commoninit
libvirt_cloudinit_disk.commoninit2
libvirt_cloudinit_disk.commoninit3
libvirt_domain.domain_vm1
libvirt_domain.domain_vm2
libvirt_domain.domain_vm3
libvirt_volume.volume-vm1
libvirt_volume.volume-vm2
libvirt_volume.volume-vm3
```

2. Kemudian apabila sudah ketemu yang menyebabkan error, hapus state dengan command:

```
terraform state rm nama-resource
```
contohnya:
```
terraform state rm libvirt_domain.domain_vm1
```
maka hasilnya akan seperti dibawah
```
libvirt_domain.domain_vm1
Removed libvirt_domain.domain_vm1
Successfully removed 1 resource instance(s).
```

3. Terakhir jalankan kembali command
```
terraform destroy
```
maka hasilnya command destroy dapat berjalan normal kembali
```
libvirt_volume.volume-vm2: Destruction complete after 0s
libvirt_cloudinit_disk.commoninit3: Destruction complete after 0s
libvirt_volume.volume-vm1: Destruction complete after 0s
libvirt_cloudinit_disk.commoninit: Destruction complete after 0s
libvirt_volume.volume-vm3: Destruction complete after 0s
libvirt_cloudinit_disk.commoninit2: Destruction complete after 0s

Destroy complete! Resources: 6 destroyed.

```
