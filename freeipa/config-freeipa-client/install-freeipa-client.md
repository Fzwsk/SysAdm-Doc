#Install freeIPA-Client

Environment

-Centos 7 /RAM 1GB/1 CPU/Disk 20GB

-FreeIPA Server

-FreeIPA Client

ubah hostname pada server

```
hostnamectl set-hostname ibra.labsadmin247.com; exec bash
```

disabled pada selinuxnya

```
sed -i 's/enforcing/disabled/g' /etc/selinux/config 
```

update dan reboot pada server

```
yum update ;reboot -y
```

tambahkan ip dan fqdn dari freeipa-server dan freeipa-client nya

```
echo "<isikan ip> <isikan fqdn>" >> /etc/hosts
```

contoh

`echo "192.168.10.137 ibra.labsadmin247.com" >> /etc/hosts`

masuk ke file /etc/resolve.conf untuk mengubah nameserver nya, ubah menjadi alamat ip dari freeipa-server

```
vi /etc/resolve.conf 
```

jika sudah lalu install freeipa-client nya 

```
 yum install -y ipa-client.x86_64 
```

configurasi ipa-client

```
ipa-client-install --mkhomedir
```
Continue to configure the system with these values? [no]: (pilih yes)

Synchronizing time with KDC...

Attempting to sync time using ntpd.  Will timeout after 15 seconds

User authorized to enroll computers: isikan sesuai dengan user login dari ui freeipa-server

Password for admin@LABSADMIN247.COM: isikan sesuai dengan pass login dari ui freeipa-server

Successfully retrieved CA cert

```
Subject:     CN=Certificate Authority,O=LABSADMIN247.COM

Issuer:      CN=Certificate Authority,O=LABSADMIN247.COM

Valid From:  2022-10-05 03:19:03

Valid Until: 2042-10-05 03:19:03
```

kemudian open port pada sisi freeipa client nya

```
firewall-cmd --add-service={http,https,dns,ntp,freeipa-ldap,freeipa-ldaps} --permanent
```

```
firewall-cmd --reload
```

lalu cek di browser dengan memanggil fqdn dari freeipa server

