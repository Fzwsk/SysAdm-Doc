# Running Logstash & Filebeat docker

## Enviroment
- Docker
- centos 7
- Docker file
## Step

1. Pertama masuk ke direktori logstash lalu 

    ```
    cat readme.txt

    ```

    isi dari file tersebut
    ```
    # running logstash

    docker run -d \
    -p 5044:5044 \
    -v $(pwd)/logstash-filebeat.conf:/opt/logstash/pipeline/logstash.conf \
    --name slb-logstash \
    slb-logstash:latest

    # running filebeat

    docker run -d \
    --name slb-beats \
    -v /var/dummy:/var/dummy \
    -v /etc/filebeat/filebeat.yml:/usr/share/filebeat/filebeat.yml:ro \
    docker.elastic.co/beats/filebeat:8.3.1

2. Lalu build image dan container logstash

    ```
    docker build -t slb-logstash:latest .

    ```

    tunggu sampai selesai lalu jalankan logstash 

    ```
    docker run -d \
    -p 5044:5044 \
    -v $(pwd)/logstash-filebeat.conf:/opt/logstash/pipeline/logstash.conf \
    --name slb-logstash \
    slb-logstash:latest
    ```
    
3. build image dan container filebeat
   
   edit file logstash-filebeat.conf pada bagian ip dan tes adalah collection menyesuaikan 

   ```
   
    output {
        solr_http {
        id => "solr_plugin_1"
        solr_url => "http://192.168.3.31:8983/solr/tes"
        }
    }

   ```
   lalu ubah permission pada  /etc/filebeat/filebeat.yml

   ```
   chmod 755 /etc/filebeat/filebeat.yml
   ```

   lalu buat dan jalankan container file beat 
   ```
   docker run -d \
    --name slb-beats \
    -v /var/dummy:/var/dummy \
    -v /etc/filebeat/filebeat.yml:/usr/share/filebeat/filebeat.yml:ro \
    docker.elastic.co/beats/filebeat:8.3.1
    ```
    edit filebeat.yml ganti localhost menjadi ip server

    ```
    output.logstash:
   # The Logstash hosts
   hosts: ["192.168.3.31:5044"]
   ```

   lalu restart logstash dan filebeat

    ```
    docker restart slb-logstash
    docker restart slb-beats
    ```

4. buka pada solr http://192.168.3.31:9901/#/dashboard pilih non-time series dashboard isi collection name lalu cek apakah log berhasil ditampilkan
