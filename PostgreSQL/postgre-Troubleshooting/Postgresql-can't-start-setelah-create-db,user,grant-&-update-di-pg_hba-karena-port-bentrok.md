# Postgresql can't start setelah create db,user,grant & update di pg_hba karena port bentrok

## Issue
```txt
● postgresql.service - PostgreSQL database server
   Loaded: loaded (/usr/lib/systemd/system/postgresql.service; disabled; vendor preset: disabled)
   Active: failed (Result: exit-code) since Mon 2023-01-30 21:40:33 WIB; 14s ago
  Process: 3590 ExecStart=/usr/bin/pg_ctl start -D ${PGDATA} -s -o -p ${PGPORT} -w -t 300 (code=exited, status=1/FAILURE)
  Process: 3584 ExecStartPre=/usr/bin/postgresql-check-db-dir ${PGDATA} (code=exited, status=0/SUCCESS)

Jan 30 21:40:32 brj2prdsmlmst.solusi247.com pg_ctl[3590]: LOG:  could not bind IPv6 socket: Address already in use
Jan 30 21:40:32 brj2prdsmlmst.solusi247.com pg_ctl[3590]: HINT:  Is another postmaster already running on port 5432? If not, ...etry.
Jan 30 21:40:32 brj2prdsmlmst.solusi247.com pg_ctl[3590]: WARNING:  could not create listen socket for "*"
Jan 30 21:40:32 brj2prdsmlmst.solusi247.com pg_ctl[3590]: FATAL:  could not create any TCP/IP sockets
Jan 30 21:40:33 brj2prdsmlmst.solusi247.com pg_ctl[3590]: pg_ctl: could not start server
Jan 30 21:40:33 brj2prdsmlmst.solusi247.com pg_ctl[3590]: Examine the log output.
Jan 30 21:40:33 brj2prdsmlmst.solusi247.com systemd[1]: postgresql.service: control process exited, code=exited status=1
Jan 30 21:40:33 brj2prdsmlmst.solusi247.com systemd[1]: Failed to start PostgreSQL database server.
Jan 30 21:40:33 brj2prdsmlmst.solusi247.com systemd[1]: Unit postgresql.service entered failed state.
Jan 30 21:40:33 brj2prdsmlmst.solusi247.com systemd[1]: postgresql.service failed.
Hint: Some lines were ellipsized, use -l to show in full.
```

## Environment

- Postgresql
- Linux OS

## Step-step Solving

1. Cek di file postgresql.service dan postgresql.conf path /usr/lib/systemd/system/postgresql.service  /var/lib/pgsql/data/postgresql.conf untuk mengetahui port dari PGDATA mengarah ke port yang mana
    ```
    search \PGDATA
    ```

2. Ubah port ke port yang tersedia, lalu Save

3. Jika sudah di ubah jalankan reload daemon, dengan perintah
    ```
    systemctl daemon-reload
    ```
4. Lalu restart postgresql

5. Coba kembali create db,user,grant & update di pg_hba

6. Restart kembali postgresql.