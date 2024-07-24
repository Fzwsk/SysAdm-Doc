# Instalasi minikube di Centos 7

# Environment

- Centos 7

- Docker

- KVM

- minikube

# Step

1. update pada sistem OS nya dan install epel-release

   ```
   yum install -y epel-release
   ```
2. Siapkan kvm dan enable & start untuk libvirtd service nya

   berikut command untuk install kvm

   ```
   yum install -y qemu-kvm qemu-img virt-manager libvirt libvirt-python libvirt-client virt-install virt-viewer bridge-utils
   ```
   selanjutnya enable dan start libvirtd nya

   ```
   systemctl enable libvirtd && systemctl start libvirtd
   ```
   kemudian install x-windows package, dan reboot jika sudah menginstall package nya 
   
   ```
   yum install "@X Window System" xorg-x11-xauth xorg-x11-fonts-* xorg-x11-utils -y
   ```
3. Kemudian siapkan docker dan enable & start docker service nya
   
   berikut command untuk install docker

   ```
   yum install -y yum-utils
   ```
   
   ```
   yum-config-manager --add-repo https://download.docker.com/linux/centos/docker-ce.repo
   ```

   ```
   yum install docker-ce docker-ce-cli containerd.io docker-compose-plugin -y
   ```

   enable & start docker

   ```
   systemctl enable docker && systemctl start docker
   ```
4. buat user baru dan jadikan user tersebut menjadi sudoers dan docker group agar dapat diakses dari libvirtd dan docker
    ```
    adduser username
    passwd username
    usermod -a -G wheel username
    ```
        note cek user sudah sudoers :
        1. pertama switch user baru
        2. kemudian cek dengan running sudo su
        3. kalau bisa masuk ke root artinya sudoers berhasil

    cara add user to docker group user
    ```
    sudo usermod -aG docker $USER
    newgrp docker
    ```
    note cek user sudah docker group :
    ```
    docker version
    ```

5. masuk ke file /etc/libivirt/libvirtd.conf dan edit konfigurasi dengan menghilangkan tanda # pada bagian, jika sudah restart libvirtd service nya

   ```
   unix_sock_group = "libvirt"
   unix_sock_rw_perms = "0770"
   ```

   ```
   systemctl restart libvirtd
   ```
6. Selanjutnya download package minikube binary ke local, dan tambahkan permission eksekusi untuk file minikube-linux-amd64

   ```
   wget https://storage.googleapis.com/minikube/releases/latest/minikube-linux-amd64
   ```

   ```
   chmod +x minikube-linux-amd64
   ```
7. pindahkan file minikube tersebut ke path /usr/local/bin/minikube
   
   ```
   mv minikube-linux-amd64 /usr/local/bin/minikube
   ```

   cek version dari minikube

   ```
   minikube version
   ```
8. Kemudian install package kubectl ke local, dan tambahkan permission eksekusi untuk file kubectl

   ```
   wget https://storage.googleapis.com/kubernetes-release/release/`curl -s https://storage.googleapis.com/kubernetes-release/release/stable.txt`/bin/linux/amd64/kubectl
   ```
   tambahkan permission ke file kubectl

   ```
   chmod +x kubectl
   ```
9. Pindahkan file kubectl ke path /usr/local/bin/kubectl dan verifikasi instalasi version untuk kubectl nya
   
   ```
   mv kubectl /usr/local/bin/kubectl
   ```

   command verifikasi instalasi version dari kubectl
   
   ```
   kubectl version --client -o json
   ```
10. Jika sudah maka perlu untuk start minikube, jika start nya menggunakan user root maka perlu parameter --force, jika tidak user root maka tanpa parameter --force
    
    ```
    minikube start --force
    ```

## Reference 
- [https://phoenixnap.com/kb/install-minikube-on-centos](https://phoenixnap.com/kb/install-minikube-on-centos)
 





   
