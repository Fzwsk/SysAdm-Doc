# Monitoring VM With Prometheus, Grafana, Node Exporter

## Environment

- VirtualBox 6.*.*
- CentOS 7

## Setup Prometheus

1. Download installer Prometheus dari official release Github menggunakan command `wget`
   
   ```bash
   wget https://github.com/prometheus/prometheus/releases/download/v2.18.1/prometheus-2.18.1.linux-amd64.tar.gz
   ```

2. Lalu extract file *tar.gz* tadi menggunakan command `tar`

   ```
   tar -xzf prometheus-2.18.1.linux-amd64.tar.gz
   ```

3. Tambahkan value dibawah di `/etc/systemd/system/prometheus.service` menggunakan command `vi` / `nano`

   ```
   [Unit]
   Description=Prometheus Server
   Wants=network-online.target
   After=network-online.target

   [Service]
   User=prometheus
   Group=prometheus
   Type=simple
   ExecStart= /home/centos/prometheus-2.18.1.linux-amd64/prometheus \
   --config.file= /home/centos/prometheus-2.18.1.linux-amd64/prometheus.yml \
   --storage.tsdb.path=/home/centos/prometheus-2.18.1.linux-amd64/ \
   --web.console.templates= /home/centos/prometheus-2.18.1.linux-amd64/consoles \
   --web.console.libraries= /home/centos/prometheus-2.18.1.linux-amd64/console_libraries

   [Install]
   WantedBy=multi-user.target
   ```

4. Start prometheus service dengan `cd` terlebih dahulu ke tempat hasil ekstrak prometheus tadi kemudian running

   ```bash
   nohup ./prometheus.sh &
   ```
5. Jangan lupa untuk menambahkan rule firewall Prometheus atau bisa juga dengan stop, dan disable service `firewalld`

## Setup Node Exporter

1. Mempersiapkan file tar.gz node exporter dari official web kemudian download menggunakan command `wget` dan tidak lupa ekstrak menggunakan `tar`

   ```
   wget https://github.com/prometheus/node_exporter/releases/download/v1.0.0-rc.1/node_exporter-1.0.0-rc.1.linux-amd64.tar.gz
   ```

   ```
   tar -xzf node_exporter-1.0.0-rc.1.linux-amd64.tar.gz
   ```

2. lalu tambahkan config dibawah `/etc/systemd/system/node_exporter.service`

   ```
   [Unit]

   Description=node_exporter
   Wants=network-online.target
   After=network-online.target

   [Service]

   User=prometheus
   Group=prometheus
   Type=simple
   ExecStart=/home/centos/node_exporter-1.0.0-rc.1.linux-amd64/node_exporter

   [Install]

   WantedBy=multi-user.target
   ```

3. reload daemon dengan command

   ```
   systemctl daemon-reload
   ```

4. ganti direktori ke folder prometheus dan edit file `prometheus.yml` sesuaikan ip dan port yang ingin dimonitoring

   ```
   - job_name: 'node_exporter'
   static_configs:
   - targets: ['localhost:9100']
   ```

## Setup Grafana

1. tambahkan value dibawah pada file yang terletak di `/etc/yum.repos.d/grafana.repo`

   ```
   [grafana]
   name=grafana
   baseurl=https://packages.grafana.com/oss/rpm
   repo_gpgcheck=1
   enabled=1
   gpgcheck=1
   gpgkey=https://packages.grafana.com/gpg.key
   sslverify=1
   sslcacert=/etc/pki/tls/certs/ca-bundle.crt
   ```

2. Install Grafana menggunakan command

   ```
   yum install -y grafana
   ```

3. start dan enable service *Grafana* dengan command

   ```
   systemctl start grafana-server; systemctl enable grafana-server
   ```

## Reference

- [https://jhooq.com/prometheous-grafan-setup/](https://jhooq.com/prometheous-grafan-setup/)
- [https://geekflare.com/prometheus-grafana-setup-for-linux/](https://geekflare.com/prometheus-grafana-setup-for-linux/)