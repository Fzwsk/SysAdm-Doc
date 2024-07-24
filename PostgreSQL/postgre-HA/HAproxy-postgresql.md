# HAproxy postgresql

## Prerequisite
- Machine: node1   IP: 10.1.80.185  Role: Postgresql, Patroni,HAproxy
- Machine: node2   IP: 10.1.80.132  Role: Postgresql, Patroni,Etcd

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
2. Install Patroni di setiap node dengan menggunakan pip3
    ```bash
    sudo yum install python3 python3-pip
    pip3 install --upgrade setuptools
    pip3 install psycopg2-binary
    pip3 install patroni
    pip3 install python-etcd
    ```
3. Install etcd di node 2 
    ```bash
    sudo yum install etcd
    ```
    lalu edit /etc/etcd/etcd.conf sesuai ip node pada konfig berikut  
    ```
    ETCD_LISTEN_PEER_URLS="http://10.1.80.132:2380"
    ETCD_LISTEN_CLIENT_URLS="http://localhost:2379,http://10.1.80.132:2379"
    ETCD_INITIAL_ADVERTISE_PEER_URLS="http://10.1.80.132:2380"
    ETCD_INITIAL_CLUSTER="default=http://10.1.80.132:2380"
    ETCD_ADVERTISE_CLIENT_URLS="http://10.1.80.132:2379"
    ETCD_INITIAL_CLUSTER_TOKEN="etcd-cluster"
    ETCD_INITIAL_CLUSTER_STATE="new"
    ```
    ```bash
    sudo systemctl restart etcd
    ``` 
4. Konfig patroni disetiap node di /etc/patroni.yml
   node 1
    ```bash
    scope: postgres-ha
    name: node1

    restapi:
        listen: 10.1.80.185:8008
        connect_address: 10.1.80.185:8008

    etcd:
        host: 10.1.80.132:2379

    bootstrap:
        dcs:
            ttl: 30
            loop_wait: 10
            retry_timeout: 10
            maximum_lag_on_failover: 1048576
            postgresql:
                use_pg_rewind: true

        initdb:
        - encoding: UTF8
        - data-checksums

        pg_hba:
        - host replication replicator 127.0.0.1/32 md5
        - host replication replicator 10.1.80.185/0 md5
        - host replication replicator 10.1.80.132/0 md5
        - host all all 0.0.0.0/0 md5

        users:
            admin:
                password: adminpass
                options:
                    - createrole
                    - createdb

    postgresql:
        listen: 10.1.80.185:5432
        connect_address: 10.1.80.185:5432
        bin_dir: /usr/pgsql-12/bin/
        data_dir: /mnt/patroni
        pgpass: /tmp/pgpass
        authentication:
            replication:
                username: replicator
                password: rep-pass
            superuser:
                username: postgres
                password: postgres-pass
        parameters:
            unix_socket_directories: '.'

    tags:
        nofailover: false
        noloadbalance: false
        clonefrom: false
        nosync: false
    ```
    node 2 
    ```bash
    scope: postgres-ha
    name: node2

    restapi:
        listen: 10.1.80.132:8008
        connect_address: 10.1.80.132:8008

    etcd:
        host: 10.1.80.132:2379

    bootstrap:
        dcs:
            ttl: 30
            loop_wait: 10
            retry_timeout: 10
            maximum_lag_on_failover: 1048576
            postgresql:
                use_pg_rewind: true

        initdb:
        - encoding: UTF8
        - data-checksums

        pg_hba:
        - host replication replicator 127.0.0.1/32 md5
        - host replication replicator 10.1.80.185/0 md5
        - host replication replicator 10.1.80.132/0 md5
        - host all all 0.0.0.0/0 md5

        users:
            admin:
                password: admin
                options:
                    - createrole
                    - createdb

    postgresql:
        listen: 10.1.80.132:5432
        connect_address: 10.1.80.132:5432
        bin_dir: /usr/pgsql-12/bin/
        data_dir: /mnt/patroni
        pgpass: /tmp/pgpass
        authentication:
            replication:
                username: replicator
                password: rep-pass
            superuser:
                username: postgres
                password: postgres-pass
        parameters:
            unix_socket_directories: '.'

    tags:
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
        server node1 10.1.80.185:5432 maxconn 100 check port 8008
        server node2 10.1.80.132:5432 maxconn 100 check port 8008
    ```
    cek haproxy.cfg lalu restart
    ```bash
    sudo /usr/sbin/haproxy -c -V -f /etc/haproxy/haproxy.cfg 
    sudo systemctl restart haproxy
    ```
    Untuk monitor bisa akses ke http://10.1.80.185:7000 dan bisa test dengan cara matikan patroni di node1 untuk mengecek node2 langsung menggantikan atau tidak









