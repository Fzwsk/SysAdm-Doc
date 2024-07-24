# HAproxy postgresql

## List Host dan Role masing-masing Host
- master  192.168.100.40 hapsqlmaster.halo.com   Role: Etcd dan HAproxy
- node1   192.168.100.42 hapsqlworker01.halo.com Role: Postgresql dan Patroni
- node2   192.168.100.59 hapsqlworker02.halo.com Role: Postgresql dan Patroni

## Step 

1. Install postgresql 12 di setiap node
    ```bash
    sudo yum install https://download.postgresql.org/pub/repos/yum/reporpms/EL-7-x86_64/pgdg-redhat-repo-latest.noarch.rpm
    sudo yum install postgresql12 postgresql12-server
    sudo /usr/pgsql-12/bin/postgresql-12-setup initdb
    ln -s /usr/lib/postgresql/12/bin/* /usr/sbin/
    systemctl enable postgresql-12
    systemctl start postgresql-12
    ```

2. Install Patroni di setiap node2 dan node2
  ```bash
  yum -y install https://download-ib01.fedoraproject.org/pub/epel/7/x86_64/Packages/p/python3-psycopg2-2.7.7-2.el7.x86_64.rpm https://github.com/cybertec-postgresql/patroni-packaging/releases/download/1.6.0-1/patroni-1.6.0-1.rhel7.x86_64.rpm
  ```

3. Copy file konfigurasi patroni di node1 dan node2
  ```bash
  cp /opt/app/patroni/etc/postgresql.yml.sample /opt/app/patroni/etc/postgresql.yml
  ```

4. Konfig patroni disetiap node di /etc/patroni.yml
   *node 1*
    ```bash
    scope: postgres
    namespace: /pg_cluster/
    name: node1

    restapi:
      listen: 192.168.100.42:8008
      connect_address: 192.168.100.42:8008

    etcd:
      host: 192.168.100.40:2379

    bootstrap:
      dcs:
      ttl: 30
      loop_wait: 10
       retry_timeout: 10
      maximum_lag_on_failover: 1048576
    postgresql:
      use_pg_rewind: true
      use_slots: true
      parameters:

    initdb:
      - encoding: UTF8
      - data-checksums

    pg_hba:
    # - host replication replicator 127.0.0.1/32 md5
    - host replication replicator 192.168.100.42/24 md5
    - host replication replicator 192.168.100.59/24 md5
    # - host replication replicator 192.168.10.28/27 md5
    - host all all 0.0.0.0/0 md5

    users:
    admin:
      password: admin
      options:
        - createrole
        - createdb

    postgresql:
      listen: 192.168.10.224:5432 # isikan-ip-vm-worker
      connect_address: 192.168.10.42:5432
      data_dir: /var/lib/pgsql/12/data
      bin_dir: /usr/pgsql-12/bin
      pgpass: /tmp/pgpass
      authentication:
    replication:
      username: replicator
      password: reppassword
    superuser:
      username: postgres
      password: postgrespassword

    categories:
      nofailover: false
      noloadbalance: false
      clonefrom: false
      nosync: false
    ```

    *node 2* 
    ```bash
    scope: postgres
    namespace: /pg_cluster/
    name: node2

    restapi:
      listen: 192.168.100.59:8008
      connect_address: 192.168.100.59:8008

    etcd:
      host: 192.168.100.40:2379

    bootstrap:
      dcs:
        ttl: 30
        loop_wait: 10
        retry_timeout: 10
        maximum_lag_on_failover: 1048576
    postgresql:
      use_pg_rewind: true
      use_slots: true
      parameters:

    initdb:
      - encoding: UTF8
      - data-checksums

    pg_hba:
    # - host replication replicator 127.0.0.1/32 md5
    - host replication replicator 192.168.100.42/24 md5
    - host replication replicator 192.168.100.59/24 md5
    # - host replication replicator 192.168.10.28/27 md5
    - host all all 0.0.0.0/0 md5

    users:
      admin:
        password: admin
        options:
          - createrole
          - createdb

    postgresql:
      listen: 192.168.100.59:5432
      connect_address: 192.168.100.59:5432
      data_dir: /var/lib/pgsql/12/data
      bin_dir: /usr/pgsql-12/bin
      pgpass: /tmp/pgpass
    authentication:
      replication:
        username: replicator
        password: reppassword
      superuser:
        username: postgres
        password: postgrespassword

    categories:
      nofailover: false
      noloadbalance: false
      clonefrom: false
      nosync: false
    ```

    Buat direktori untuk data_dir
    ```bash 
    sudo mkdir -p /mnt/patroni
    sudo chown postgres:postgres /data/patroni
    sudo chmod 700 /data/patroni
    ```
    Buat service patroni
    ```bash
    sudo nano /etc/systemd/system/patroni.service
    ```
    isikan berikut 
    ```bash
    [Unit]
    Description=High availability PostgreSQL Cluster
    After=syslog.target network.target
    [Service]
    Type=simple
    User=postgres
    Group=postgres
    ExecStart=/usr/local/bin/patroni /etc/patroni.yml
    KillMode=process
    TimeoutSec=30
    Restart=no

    [Install]
    WantedBy=multi-user.target
    ```

5. Sebelum start patroni aktifkan watchdog support disetiap node
    ```bash
    sudo modprobe softdog
    sudo chown postgres /dev/watchdog
    ```
    lalu jalankan patroni 
    ```bash
    systemctl start patroni
    systemctl status patroni
    ```
    untuk mengecek kedua node bisa dengan 
    ```bash
    patronictl -c /etc/patroni.yml list
    ```


6. Install Haproxy di node 1 
    ```bash 
    sudo yum install haproxy
    sudo vi /etc/haproxy/haproxy.cfg
    ```
    lalu edit sbg berikut
    ```
    global
    maxconn 100

    defaults
        log global
        mode tcp
        retries 2
        timeout client 30m
        timeout connect 4s
        timeout server 30m
        timeout check 5s

    listen stats
        mode http
        bind *:7000
        stats enable
        stats uri /
        stats refresh 10s

    listen postgres
        bind *:5000
        option httpchk
        http-check expect status 200
        default-server inter 3s fall 3 rise 2 on-marked-down shutdown-sessions
        server node1 192.168.100.42:5432 maxconn 100 check port 8008
        server node2 192.168.100.59:5432 maxconn 100 check port 8008
    ```
    cek haproxy.cfg lalu restart
    ```bash
    sudo /usr/sbin/haproxy -c -V -f /etc/haproxy/haproxy.cfg 
    sudo systemctl restart haproxy
    ```
## testing connection
connect dengan perintah dibawah dengan password `postgrespassword`
`psql -h 192.168.100.42 -p 5000 -U postgres`

## lalu cek status dari postgres yang HA
cek di url login dengan `username:password` 
`http://192.168.100.40:2233/`