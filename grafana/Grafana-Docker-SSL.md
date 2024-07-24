# Grafana Docker SSL

## Step 

1. Buat direktori grafana
    ```bash
    mkdir grafana && cd grafana
2. Generate .key & .crt
    ```bash
    openssl genrsa -out grafana.key 2048
    openssl req -new -key grafana.key -out grafana.csr
    openssl x509 -req -days 365 -in grafana.csr -signkey grafana.key -out grafana.crt
    ```
3. Buat docker-compose.yml yang isinya
    ```
    version: '3'
    services:
        grafana:
            image: 'grafana/grafana:8.5.6'
            ports:
                - '3000:3000'
            environment:
                - VIRTUAL_PROTO=https
                - GF_SERVER_CERT_KEY=/etc/grafana/grafana.key
                - GF_SERVER_CERT_FILE=/etc/grafana/grafana.crt
                - GF_SERVER_PROTOCOL=https
            volumes:
                - ./grafana.key:/etc/grafana/grafana.key
                - ./grafana.crt:/etc/grafana/grafana.crt
    ```
4. Jalankan docker-compose up

*note apabila saat menjalankan docker-compose up muncul error /etc/grafana/  permission denied masuk ke container dan chmod +r /etc/grafana ato cek permission direktory
