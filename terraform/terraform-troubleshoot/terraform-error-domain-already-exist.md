# Terraform error domain already exist with uuid

## Issue
```
Error defining libvirt domain: virError(Code=9, Domain=20, Message='operation failed: domain 'terraform-kvm-ansible' already exists with uuid 939fb3d6-52f8-4855-8c78-e8d98bffc6e6
```
## Environment
- Fedora 35
- Terraform v1.2.6

## Resolution
1. Cek semua vm yang telah dibuat di terraform dengan command

```
sudo virsh list --all
```

```
[fzwsk@fedora teraform]$ sudo virsh list --all
[sudo] password for fzwsk: 
 Id   Name                    State
----------------------------------------
 -    centos7.0               shut off
 -    centos7.0-2             shut off
 -    node-01                 shut off
 -    node-02                 shut off
 -    node-03                 shut off
 -    terraform-kvm-ansible   shut off
 -    ubuntu18.04             shut off
 -    ubuntu18.04-clone       shut off
```

2. Kemudian hapus node yang sudah exist di pesan error, dengan run command
```
sudo virsh undefine name
```
contohnya
```
[fzwsk@fedora teraform]$ sudo virsh undefine node-01
Domain 'node-01' has been undefined

[fzwsk@fedora teraform]$ sudo virsh undefine node-02
Domain 'node-02' has been undefined

[fzwsk@fedora teraform]$ sudo virsh undefine node-03
Domain 'node-03' has been undefined
```

3. Selanjutnya cek kembali dengan running ulang
```
sudo virsh list --all
```

```
$ sudo virsh list --all

 Id   Name                    State
----------------------------------------
 -    centos7.0               shut off
 -    centos7.0-2             shut off
 -    terraform-kvm-ansible   shut off
 -    ubuntu18.04             shut off
 -    ubuntu18.04-clone       shut off
```

4. Terakhir jalankan kembali terraform apply, maka apabila nama domain sama tidak akan muncul error domain exist
