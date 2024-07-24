#Replica freipa server dari sisi freeipa client

Environment

- Centos 7/RAM 2GB/CPU 1/DISK 20GB

- FreeIPA Client

Pastikan FreeIPA Client sudah terinstall dan terkonfigurasi di Centos 7, jika sudah maka bisa untuk melanjutkan ke replica freeipa server

Berikut langkah-langkah untuk replica freeipa server di dalam freeipa client;

install ipa-server terlebih dahulu di freeipa clientnya

```
yum install -y ipa-server
```
setelah itu lakukan kinit admin pada sisi freeipa client maupun server

```
kinit admin
```
selanjutnya add hostgroup-member dari host freeipa client, pastikan dalam menambahkannya di dalam freeipa server, berikut command untuk add hostgroup-member nya

```
ipa hostgroup-add-member <isikan nama hostgroup> --hosts <isikan fqdn dari freeipa client>
```
kemudian ubah dns yang ada di file /etc/resolve.conf, untuk mengubahnya dilakukan di dalam freeipa server

ubah pada bagian nameserver diisi sesuai dengan ip dari freeipa server

selanjutnya tambahkan ip dan fqdn dari freeipa client ke file /etc/hosts milik freeipa server

```
echo "<isikan ip dari freeipa client> <isikan fqdn dari freeipa client>" >> /etc/hosts
```
kemudian cek domain dari freeipa server maupun dari sisi freeipa clientnya, tujuannya untuk mengetahui apakah sudah terdeteksi

```
nslookup <isikan fqdn dari freeipa server>
```

```
nslookup <isikan fqdn dari freeipa client>
```

setelah itu lakukan install replica untuk freeipa server nya, pastikan dalam running command nya dilakukan di freeipa client

```
ipa-replica-install
```
jika muncul notice please check your DNS setup dan pada bagina continue pilih opsi yes

dan proses replica akan terinstall dan melakukan clone dari freeipa server 

lalu tinggal stop firewalld nya agar dapat di akses dari browser local, maka stop firewall nya

```
systemctl stop firewalld
```

 