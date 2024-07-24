# Install freeipa

Environment

-Centos 7 /RAM 2GB/1 CPU/DISK 20GB

-Freeipa Server

1.set hostname pada server

```
hostnamectl set-hostname <isikanhostnamenya>
```
```
exec bash
```
2.nonaktifkan selinux

```
sed -i 's/enforcing/disabled/g' /etc/selinux/config
```
3.update pada server dan reboot

```
yum update -y ;reboot
```
4.tambahkan ip dan fqdn untuk freeipa server di file /etc/hosts

```
echo "<tambahkan ip dan fqdn>" >> /etc/hosts
```
```
cat /etc/hosts
```
5.install freeipa server beserta dns nya

```
yum install ipa-server ipa-server-dns -y
```
config freeipa

```
ipa-server-install
```

```

The log file for this installation can be found in /var/log/ipaserver-install.log
==============================================================================
This program will set up the IPA Server.

This includes:
* Configure a stand-alone CA (dogtag) for certificate management
* Configure the Network Time Daemon (ntpd)
* Create and configure an instance of Directory Server
* Create and configure a Kerberos Key Distribution Center (KDC)
* Configure Apache (httpd)
* Configure the KDC to enable PKINIT

To accept the default shown in brackets, press the Enter key.

Do you want to configure integrated DNS (BIND)? [no]: <pilih yes>

Enter the fully qualified domain name of the computer
on which you're setting up server software. Using the form
<hostname>.<domainname>
Example: master.example.com.


Server host name [ade.labsadmin247.com]:<enter saja krn dipastikan sudah sama>

Warning: skipping DNS resolution of host ade.labsadmin247.com
The domain name has been determined based on the host name.

Please confirm the domain name [labsadmin247.com]:<enter saja>

The kerberos protocol requires a Realm name to be defined.
This is typically the domain name converted to uppercase.

Please provide a realm name [LABSADMIN247.COM]: <enter saja>
Certain directory server operations require an administrative user.
This user is referred to as the Directory Manager and has full access
to the Directory for system management tasks and will be added to the
instance of directory server created for IPA.
The password must be at least 8 characters long.

Directory Manager password: <isikan pass untuk freeipa>
Password (confirm): <isikan pass untuk freeipa>

The IPA server requires an administrative user, named 'admin'.
This user is a regular system account used for IPA server administration.

IPA admin password:<isikan pass untuk freeipa>
Password (confirm):<isikan pass untuk freeipa>

jika berhasil maka akan muncul seperti dibawah ini 

Checking DNS domain labsadmin247.com., please wait ...
Do you want to configure DNS forwarders? [yes]: <pilih yes >
Following DNS servers are configured in /etc/resolv.conf: 192.168.10.1
Do you want to configure these servers as DNS forwarders? [yes]: <pilih yes>
All DNS servers from /etc/resolv.conf were added. You can enter additional addresses now:
Enter an IP address for a DNS forwarder, or press Enter to skip:
Checking DNS forwarders, please wait ...
DNS server 192.168.10.1: answer to query '. SOA' is missing DNSSEC signatures (no RRSIG data)
Please fix forwarder configuration to enable DNSSEC support.
(For BIND 9 add directive "dnssec-enable yes;" to "options {}")
WARNING: DNSSEC validation will be disabled
Do you want to search for missing reverse zones? [yes]: <pilih no>

<nah pada bagian ini akan menampilkan informasi mengenai hostname,ip server,dan domain> 

The IPA Master Server will be configured with:
Hostname:       ade.labsadmin247.com
IP address(es): 192.168.10.237
Domain name:    labsadmin247.com
Realm name:     LABSADMIN247.COM

BIND DNS server will be configured to serve IPA domain with:
Forwarders:       192.168.10.1
Forward policy:   only
Reverse zone(s):  No reverse zone

Continue to configure the system with these values? [no]: <pilih yes>

The following operations may take some minutes to complete.
Please wait until the prompt is returned.
```
jika dalam konfigurasi di freeipa sudah selesai maka cek status dari firewall yang ada di local

```
systemctl status firewalld
```
open port pada firewalld agar ui freeipa dapat diakses di browser windows bukan browser local

```
firewall-cmd --add-service={http,https,dns,ntp,freeipa-ldap,freeipa-ldaps} --permanent
```
```
firewall-cmd --reload
```
# setup openssl di freeipa nya

kemudian masuk ke winscp,dan pindahkan file ca.crt ke windows

file ca.crt itu ada di /etc/ipa/ca.crt

lalu pindahkan file ipa nya ke folder dari windows

jika sudah maka masuk ke browser, dan ketikkan domain freeipa nya

`untuk loginnya menggunakan user/pass admin/isikanpasssesuaiygdiipa-server-install`

lalu masuk ke setting,untuk mengimport certificate ke dalam browser,agar dari browser tidak mengenali certifikat yang dimiliki oleh domainnya

begitu juga dari certificat yang dimiliki oleh domain freeipanya mengenal browsernya tetapi tidak untuk browsernya

nah certificate ini agar domainnya berstatus https, agar domainnya sudah secure

jika sudah masuk ke ui dari freeipa nya, maka tinggal buka tab lain dan masuk ke 

`setting>privacy&security>certificate pilih view certificate>lalu pilih import untuk mengimport file ca.crt nya`

jika sudah tinggal refresh halaman ui freeipa nya



 



