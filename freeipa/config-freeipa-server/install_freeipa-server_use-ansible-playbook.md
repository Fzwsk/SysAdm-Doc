# install freeipa server melalui ansible playbook

Environment

-centos 7 / RAM 3GB/2 CPU/Disk 20GB

-freeIPA

1.Set hostname pada server

```
hostnamctl set-hostname <isikannamahostname>
```

```
exec bash
```

2.Tambahkan ip dan fqdn untuk freeipa server nya ke dalam file /etc/hosts

```
echo "<isikan ip dari fqdn> <isikan fqdn untuk freeipa server>" >> /etc/hosts
```

3.Jika sudah ditambahkan cek di file /etc/resolve.conf nya, pastikan nameservernya sudah sesuai dengan gateway dari dhcpnya

```
cat /etc/resolve.conf
```

4.Jika sudah install epel-release dan install ansible nya

```
yum install -y epel-release; yum install -y ansible
```
5.kemudian buat playbook.yml nya di server, lalu tambahkan isi konfigurasi seperti yang ada di bawah ini;

```
vi playbook.yml
```

```
---
# Ansible Playbook install Freeipa Server and its requirement in Localhost
- hosts : localhost
become: yes
become_user: root
become_method: sudo  
tasks :
  - name: Install Epel-Release
    become: true
    yum:
      name: epel-release
      state: latest
  - name: Running yum update
    yum:
      name: '*'
      state: latest
  - name: Disable SElinux
    selinux:
      state: disabled
  - name: Install FreeIPA Server
    become: true
    yum:
      name: ipa-server, ipa-server-dns
      state: latest
  - name: Install FreeIPA Server with DNS
    shell: ipa-server-install --realm=<isikanNAMADOMAIN> --domain=<isikannamadomain> --ds-password=<tambahkan pass untuk directory managerpassword> --admin-password=<tambahkan pass untuk login ke ui freeipa server> --mkhomedir --ssh-trust-dns --setup-dns --unattended --auto-forwarders
  - name: permit traffic in default zone for freeipa service
    firewalld:
      service: "{{ item }}"
      permanent: yes
      state: enabled
    with_items:
    - http
    - https
    - dns
    - ntp
    - freeipa-ldap
    - freeipa-ldaps
  - name: Reload config Firewall
    become: true
    shell: firewall-cmd --reload
```

6.jika sudah maka tinggal running playbook nya agar ansible melakukan automasi install dan configurasi freeipa server

```
ansible-playbook <isikan nama playbook yang sudah dibuat>.yml
```
7.jika berhasil prosesnya maka akan muncul seperti ini ;

```
[WARNING]: provided hosts list is empty, only localhost is available. Note that the implicit localhost does not match 'all'

PLAY [localhost] *********************************************************************************************************************************************

TASK [Gathering Facts] ***************************************************************************************************************************************
ok: [localhost]

TASK [Install Epel-Release] **********************************************************************************************************************************
ok: [localhost]

TASK [Running yum update] ************************************************************************************************************************************
ok: [localhost]

TASK [Disable SElinux] ***************************************************************************************************************************************
ok: [localhost]

TASK [Install FreeIPA Server] ********************************************************************************************************************************
ok: [localhost]

TASK [Install FreeIPA Server with DNS] ***********************************************************************************************************************
changed: [localhost]

TASK [permit traffic in default zone for freeipa service] ****************************************************************************************************
changed: [localhost] => (item=http)
changed: [localhost] => (item=https)
changed: [localhost] => (item=dns)
changed: [localhost] => (item=ntp)
changed: [localhost] => (item=freeipa-ldap)
changed: [localhost] => (item=freeipa-ldaps)

TASK [Reload config Firewall] ********************************************************************************************************************************
changed: [localhost]

PLAY RECAP ***************************************************************************************************************************************************
localhost                  : ok=8    changed=3    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0

[root@ade ~]#
```

8.kemudian untuk mengecek apakah sudah terinstall freeipa servernya, tinggal panggil fqdn nya dari terminal

```
curl <isikan fqdn nya>
```

9.selanjutnya jika muncul seperti yg ada dibawah ini , tinggal copy domain nya ;

```
<!DOCTYPE HTML PUBLIC "-//IETF//DTD HTML 2.0//EN">
<html><head>
<title>301 Moved Permanently</title>
</head><body>
<h1>Moved Permanently</h1>
<p>The document has moved <a href="https://ade.labs247admin.com/ipa/ui">here</a>.</p>
</body></html>
```

10.panggil ulang contohnya

```
curl  ade.labs247admin.com/ipa/ui
```
11.selanjutnya akan menampilkan halaman isi dari file html dan menampilkan bahwa domain dari freeipa server sudah bisa diakses

jika muncul seperti yang ada dibawah maka sudah pasti ui freeipa server sudah dapat diakses ;

jika didalam elemen body sudah menampilkan status enabled seperti yang ada dibawah ini, maka artinya fqdn dari server sudah bisa diakses

`<noscript>This application requires JavaScript enabled.</noscript>`




