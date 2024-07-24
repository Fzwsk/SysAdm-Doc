# Running Logstash, Filebeat, Banana With Docker

## Environment

- Logstash:latest
- Filebeat 8.3.1
- Banana:latest
- Docker

## Persiapkan folder khusus image docker yang akan di custom

1. Buka terminal, masuk ke folder dan jalankan command
    ```
    docker build -t slb-logstash:latest .
    ```

2. Tunggu hingga proses install dan patching selesai
3. Setelah proses instalasi dan patching selesai buka file *logstash.conf* kemudian ubah value ip sesuai dengan ip yang kita pakai
    ```
    input {
     beats {
      port => 5044
        }
    }

    filter {
    if "dummy" in [fields][type] {
    mutate {
       remove_field => ["log","ecs","input","tags","fields","agent" ,"os"]
       update => {
        "event" => "%{[event][original]}"
        "host" => "%{[host][hostname]}"
        }
    }
    grok {
      match => { "message" => "%{SYSLOGTIMESTAMP:syslog_timestamp} %{SYSLOGHOST:syslog_hostname} %{DATA:syslog_program}(?:\[%{POSINT:syslog_pid}\])?: %{GREEDYDATA:syslog_message}" }
    }
    date {
      match => [ "syslog_timestamp", "MMM  d HH:mm:ss", "MMM dd HH:mm:ss" ]
          }
        }
    }

    output {
      solr_http {
      id => "solr_plugin_1"
      solr_url => "http://192.168.3.29:8983/solr/tes"
        }
    }

    ```
4.  Selanjutnya jalankan perintah berikut untuk menjalankan image *Logstash*
    ```
    # running logstash

    docker run -d \
    -p 5044:5044 \
    -v $(pwd)/logstash-filebeat.conf:/opt/logstash/pipeline/logstash.conf \
    --name slb-logstash \
    slb-logstash:latest

5. Buka file *filebeat.yml* dan ubah beberapa value menjadi seperti dibawah, sesuaikan dengan ip yang di pakai masing"
    ```
    # ------------------------------ Logstash Output -------------------------------
    output.logstash:
    # The Logstash hosts
    hosts: ["192.168.3.29:5044"]

    ```
6. Lalu jalankan image filebeat dengan perintah
    ```
    docker run -d \
    --name slb-beats \
    -v /var/dummy:/var/dummy \
    -v /etc/filebeat/filebeat.yml:/usr/share/filebeat/filebeat.yml:ro \
    docker.elastic.co/beats/filebeat:8.3.1
    ```

7. Terakhir cek di Solr UI di *http:192.168.3.29:8983* / sesuai dengan ip masing"
8. Cek juga di banana ui, pastikan sudah menjalankan image banana, kemudian lakukan import sesuai dengan collection yang telah dipakai di solr 