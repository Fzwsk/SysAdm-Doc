running basic command openssl secara manual 

##Environment

-Centos 7/RAM 2GB/CPU 1
-Nginx

1. Install epel-release repo

   ```
   sudo yum install epel-release -y
   ```
2. Install nginx

   ```
   sudo yum install -y nginx
   ```
3. Disable apache first

   ```
   sudo systemctl stop httpd && sudo systemctl disable httpd
   ```
4. start and enable nginx

   ```
   sudo systemctl start nginx && sudo systemctl enable nginx
   ```
5. setup certificate for https

   ```
   mkdir -p self-managed-ca/{certs,keys}
   ```
6. masuk ke dir self-managed-ca

   ```
   cd self-managed-ca
   ```
7. buat file ca.cnf dan konfigurasikan isi file seperti yang ada dibawah;

   ```
   # OpenSSL CA configuration file
   [ ca ]
   default_ca = CA_default

   [ CA_default ]
   default_days = 365
   database = index.txt
   serial = serial.txt
   default_md = sha256
   copy_extensions = copy
   unique_subject = no

   # Used to create the CA certificate.
   [ req ]
   prompt=no
   distinguished_name = distinguished_name
   x509_extensions = extensions

   [ distinguished_name ]
   organizationName = Cockroach
   commonName = Cockroach CA

   [ extensions ]
   keyUsage = critical,digitalSignature,nonRepudiation,keyEncipherment,keyCertSign
   basicConstraints = critical,CA:true,pathlen:1

   # Common policy for nodes and users.
   [ signing_policy ]
   organizationName = supplied
   commonName = optional

   # Used to sign node certificates.
   [ signing_node_req ]
   keyUsage = critical,digitalSignature,keyEncipherment
   extendedKeyUsage = serverAuth,clientAuth

   # Used to sign client certificates.
   [ signing_client_req ]
   keyUsage = critical,digitalSignature,keyEncipherment
   extendedKeyUsage = clientAuth
   ```
   
   -jika sudah maka tinggal save 

   -lalu Membuat CA key di folder self-managed-ca

   ```
   openssl genrsa -out keys/ca.key 4096
   ```
   -tambahkan permission read ke file ca.key

   ```
   # chmod 400 keys/ca.key
   ```
   -Membuat CA certificate

   ```
   openssl req \
   -new -x509 \
   -config ca.cnf \
   -key keys/ca.key \
   -out certs/ca.crt \
   -days 365 \
   -batch
   ```
   -Reset Database
  
   ```
   rm -f index.txt serial.txt
   ```
   ```
   touch index.txt
   ```
   ```
   echo '01' > serial.txt
   ```
   -Membuat key node
   
   ```
   read -r -p "Please enter your IP Address " ipadd
   ```
   ```
   read -r -p "Please enter your hostname " hostname
   ```
   -membuat file node.cnf di folder self-managed-ca dan konfigurasikan isi file  seperti yang ada di bawah;

   ```
   # OpenSSL node configuration file
   [ req ]
   prompt=no
   distinguished_name = distinguished_name
   req_extensions = extensions

   [ distinguished_name ]
   organizationName = Cockroach

   [ extensions ]
   subjectAltName = critical,DNS:"$hostname",DNS:"$hostname",IP:"$ipadd"
   ```
   -jika sudah maka tinggal save 

   -Membuat key node di folder self-managed-ca 

   ```
   openssl genrsa -out certs/"$hostname".key 4096
   ```
   -tambahkan permission read untuk file hostname.key 

   ```
   chmod 400 certs/"$hostname".key
   ```
   -Membuat CSR untuk node(dibuat di dalam folder self-managed-ca)

   ```
   openssl req \
   -new \
   -config node.cnf \
   -key certs/$hostname.key \
   -out $hostname.csr \
   -batch
   ```
   -Menandatangani certificate

   ```
   openssl ca \
   -config ca.cnf \
   -keyfile keys/ca.key \
   -cert certs/ca.crt \
   -policy signing_policy \
   -extensions signing_node_req \
   -in $hostname.csr \
   -out certs/$hostname.crt \
   -outdir certs/ \
   -batch
   ```
   -memastikan nilai subject alter mode (verifikasi certificate, pastikan dalam runningnya masih di dalam folder self-managed-ca )

   ```
   openssl x509 -in certs/$hostname.crt -text | grep "X509v3 Subject Alternative Name" -A 1
   ```
   -note = command yang ada angka 1 - 8 adalah command yang perlu di add ke nginx.conf bila sudah verif certif

   -input file ke config nginx.conf

   -masuk ke folder /etc/nginx/ dan masuk ke file nginx.conf lalu edit isi dari file configurasinya sesuai yang sudah ada di "note"

   ```
   # For more information on configuration, see:
   #   * Official English Documentation: http://nginx.org/en/docs/
   #   * Official Russian Documentation: http://nginx.org/ru/docs/

   user nginx;
   worker_processes auto;
   error_log /var/log/nginx/error.log;
   pid /run/nginx.pid;

   # Load dynamic modules. See /usr/share/doc/nginx/README.dynamic.
   include /usr/share/nginx/modules/*.conf;

   events {
    worker_connections 1024;
   }

   http {
    log_format  main  '$remote_addr - $remote_user [$time_local] "$request" '
                      '$status $body_bytes_sent "$http_referer" '
                      '"$http_user_agent" "$http_x_forwarded_for"';

    access_log  /var/log/nginx/access.log  main;

    sendfile            on;
    tcp_nopush          on;
    tcp_nodelay         on;
    keepalive_timeout   65;
    types_hash_max_size 4096;

    include             /etc/nginx/mime.types;
    default_type        application/octet-stream;

    # Load modular configuration files from the /etc/nginx/conf.d directory.
    # See http://nginx.org/en/docs/ngx_core_module.html#include
    # for more information.
    include /etc/nginx/conf.d/*.conf;

    server {
        listen       80 default_server;
        listen       [::]:80 default_server;
     1. server_name  $ipadd;
     2. return 301 https://$host$request_uri; #redirect ke https
        root         /usr/share/nginx/html;

        # Load configuration files for the default server block.
        include /etc/nginx/default.d/*.conf;
     3. location / {
        }

        error_page 404 /404.html;
            location = /404.html {
        }

        error_page 500 502 503 504 /50x.html;
            location = /50x.html {
        }
    }

   # Settings for a TLS enabled server.

    server {
        listen       443 ssl http2;
        listen       [::]:443 ssl http2;
     4. server_name  $ipadd;
        root         /usr/share/nginx/html;

     5. ssl_certificate "/root/self-managed-ca/certs/$hostname.crt";
     6.   ssl_certificate_key "/root/self-managed-ca/certs/$hostname.key";
        ssl_session_cache shared:SSL:1m;
        ssl_session_timeout  10m;
        ssl_ciphers HIGH:!aNULL:!MD5;
        ssl_prefer_server_ciphers on;
     7. ssl_protocols TLSv1 TLSv1.1 TLSv1.2;

        # Load configuration files for the default server block.
        include /etc/nginx/default.d/*.conf;

8. location 
```
/ {
        }

        error_page 404 /404.html;
            location = /40x.html {
        }

        error_page 500 502 503 504 /50x.html;
            location = /50x.html {
        }
    }

}

```
-check nginx

```
nginx -t
```
-restart nginx

```
sudo systemctl restart nginx
```
-Disable SElinux

```
sed -i 's/enforcing/disabled/g' /etc/selinux/config
```
-reboot

```
reboot
```





