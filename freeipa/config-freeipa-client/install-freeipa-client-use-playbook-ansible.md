## Install freeipa client with playbook-ansible

## Environment

- Centos 7/RAM 2GB/CPU 2/DISK 20GB

- Freeipa Server 

- Freeipa Client

1. set hostname dari local nya

   ```
   hostnamectl set-hostname isikanhostnameyangmaudiubah \
   exec bash
   ```
2. tambahkan ip dan fqdn dari freeipa server dan dari local(tempat yang bakal dijadikan as freeipa client) ke file /etc/hosts

   ```
   echo "isikanip isikanfqdn" >> /etc/hosts
   ```
   begitu juga yang dari freeipa server, menambahkan ip dan fqdn dari freeipa client ke file /etc/hosts milik freeipa server

3. ubah value dari nameserver yang ada di file /etc/resolve.conf , sesuaikan dengan ip dari freeipa server untuk valuenya

   ```
   vi /etc/resolve.conf
   ``` 
4. Kemudian lakukan install epel-release dan ansible

   ```
   yum install -y epel-release; yum install -y ansible
   ```
5. kemudian jika sudah maka buat playbook nya
   
   ```
   vi (isikannamaplaybook).yml
   ```
   berikut isi dari playbook untuk freeipa client
   
   ```
   # Playbook install FreeIPA Client
- name: Playbook Configure IPA Clients with username / password
  hosts: localhost
  become: true

  tasks:
    - name: Install Epel-Release
      become: true
      yum:
        name: epel-release
        state: latest
    - name: Install Firewalld
      become: true
      yum:
        name: firewalld
        state: latest
    - name: Disable SElinux
      selinux:
        state: disabled
    - name: install bind9
      yum:
        name: bind-utils
        state: latest
    - name: install IPA client Package
      yum:
        name: freeipa-client
        state: latest
    - name: Configure IPA Client 
      shell: ipa-client-install --server=isikanfqdndarifreeipaserver --domain=isikandomain --realm=ISIKANREALME(domainpakehurufbesar) --principal=admin --password=isikanpass --mkhomedir --force-ntpd --unattended
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

6. jika sudah maka cek di ui nya freeipa server, jangan lupa untuk kinit admin di sisi freeipa server maupun client

   cek di menu hosts yang ada di ui nya, jika fqdn dari freeipa client muncul, maka sudah dipastikan install freeeipa client berhasil

