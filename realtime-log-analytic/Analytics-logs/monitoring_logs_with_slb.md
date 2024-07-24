# Monitoring logs SLB 
Monitoring logs with solr,logstash,filebeat & banana

## Environment
- Docker 
- logstash
- banana 
- solr
- filebeat

## Step

1. Edit filebeat.yml pada filebeat inputs isikan path log dan buat tags 

    ```
    filebeat.inputs:

    - type: log
    enabled: true
    paths:
        - /var/log/storm/workers-artifacts/crawl-mediaonline-*/*/worker.log
        - /var/log/storm/workers-artifacts/medan-twitter-crawler-*/*/worker.log
        - /var/log/storm/workers-artifacts/youtube-*/*/worker.log
    fields:
        type: storm
    multiline.type: pattern
    multiline.pattern: '^202'
    multiline.negate: true
    multiline.match: after
    multiline.max_lines: 30

    - type: log
    enabled: true
    paths:
        - "/var/log/kafka/controller.log"
    tags: [kafkacontroller]
    multiline.type: pattern
    multiline.pattern: '^\[202'
    multiline.negate: true
    multiline.match: after
    multiline.max.lines: 30

    - type: log
    enabled: true
    paths:
        - "/var/log/kafka/log-cleaner.log"
    tags: [kafkalogcleaner]
    multiline.type: pattern
    multiline.pattern: '^\[202'
    multiline.negate: true
    multiline.match: after
    multiline.max.lines: 30

    - type: log
    enabled: true
    paths:
        - "/var/log/kafka/server.log"
    tags: [kafkaserver]
    multiline.type: pattern
    multiline.pattern: '^\[202'
    multiline.negate: true
    multiline.match: after
    multiline.max.lines: 30

    - type: log
    enabled: true
    paths:
        - "/var/log/kafka/state-change.log"
    tags: [kafkastatechange]

    - type: log
    enabled: true
    paths:
        - "/var/log/kafka/ranger_kafka.log"
    tags: [kafkaranger]
    multiline.type: pattern
    multiline.pattern: '^202'
    multiline.negate: true
    multiline.match: after
    multiline.max.lines: 30

    - type: log
    enabled: true
    paths:
        - "/var/log/solr/solr.log*"
    tags: [solr]
    multiline.type: pattern
    multiline.pattern: '^202'
    multiline.negate: true
    multiline.match: after
    multiline.max.lines: 30

    - type: log
    enabled: true
    paths:
        - "/hadoop/lampp/httpd-2.4/logs/access_log"
    tags: [httpdaccesslog]

    - type: log
    enabled: true
    paths:
        - "/hadoop/lampp/httpd-2.4/logs/error_log"
    tags: [httpderrorlog]
    multiline.type: pattern
    multiline.pattern: '^\['
    multiline.negate: true
    multiline.match: after
    multiline.max.lines: 30

    - type: log
    enabled: true
    paths:
        - "/home/medan/medan2/services/redis/redis-cluster-*.log"
    tags: [redis]

    ```

    untuk multiline pattern berikut penjelasan nya https://www.elastic.co/guide/en/beats/filebeat/current/multiline-examples.html 

2. Edit file logstash-filebeat.conf masukkan grok pattern yang match untuk setiap logs 

    ```
        input {
    beats {
        port => 5044
    }
    }

    filter {
    if "storm" in [fields][type] {
        mutate {
        remove_field => ["log","ecs","input","tags","agent" ,"os"]
        update => {
            "event" => "%{[event][original]}"
            "host" => "%{[host][hostname]}"
            "fields" => "%{[fields][type]}"
            }
        }
        grok {
        break_on_match => false
        match => { "message" => "^%{TIMESTAMP_ISO8601:time} %{JAVACLASS:class} %{GREEDYDATA:method} \[%{LOGLEVEL:loglevel}\]%{DATA:message}({({[^}]+},?\s*)*})?\s*$(?<stacktrace>(?m:.*))?" }
        }
    }
    else if "kafkacontroller" in [tags] {
        mutate {
        update => {
            "tags" => "kafkacontroller"
            "event" => "%{[event][original]}"
            "host" => "%{[host][hostname]}"
        }
        remove_field => ["log","ecs","input","fields","agent" ,"os"]
        }
        grok {
        match => { "message" => "^\[%{TIMESTAMP_ISO8601:logtime}\]%{SPACE}%{LOGLEVEL:level}%{SPACE}\[(?<logger>[^\]]+)\]%{SPACE}%{GREEDYDATA:log_message}" }
        }
    }
    else if "kafkalogcleaner" in [tags] {
        mutate {
        update => {
            "tags" => "kafkalogcleaner"
            "event" => "%{[event][original]}"
            "host" => "%{[host][hostname]}"
        }
        remove_field => ["log","ecs","input","fields","agent" ,"os"]
        }
        grok {
        match => { "message" => "^\[%{TIMESTAMP_ISO8601:logtime}\]%{SPACE}%{LOGLEVEL:level}%{SPACE}%{GREEDYDATA:log_message}" }
        }
    }
    else if "kafkaserver" in [tags] {
        mutate {
        update => {
            "tags" => "kafkaserver"
            "event" => "%{[event][original]}"
            "host" => "%{[host][hostname]}"
        }
        remove_field => ["log","ecs","input","fields","agent" ,"os"]
        }
        grok {
        match => { "message" => "^\[%{TIMESTAMP_ISO8601:logtime}\]%{SPACE}%{LOGLEVEL:level}%{SPACE}%{GREEDYDATA:log_message}" }
        }
    }
    else if "kafkastatechange" in [tags] {
        mutate {
        update => {
            "tags" => "kafkastatechange"
            "event" => "%{[event][original]}"
            "host" => "%{[host][hostname]}"
        }
        remove_field => ["log","ecs","input","fields","agent" ,"os"]
        }
        grok {
        match => { "message" => "^\[%{TIMESTAMP_ISO8601:logtime}\]%{SPACE}%{LOGLEVEL:level}%{SPACE}\[(?<logger>[^\]]+)\]%{SPACE}%{GREEDYDATA:log_message}" }
        }
    }
    else if "kafkaranger" in [tags] {
        mutate {
        update => {
            "tags" => "kafkaranger"
            "event" => "%{[event][original]}"
            "host" => "%{[host][hostname]}"
        }
        remove_field => ["log","ecs","input","fields","agent" ,"os"]
        }
        grok {
        match => { "message" => "%{TIMESTAMP_ISO8601:logtime}%{SPACE}%{LOGLEVEL:level}%{SPACE}\[(?<logger>[^\]]+)\]%{SPACE}%{GREEDYDATA:log_message}$(?<stacktrace>(?m:.*))?" }
        }
    }
    else if "solr" in [tags] {
        mutate {
        update => {
            "tags" => "solr"
            "event" => "%{[event][original]}"
            "host" => "%{[host][hostname]}"
        }
        remove_field => ["log","ecs","input","fields","agent" ,"os"]
        }
        grok {
        match => { "message" => "^%{TIMESTAMP_ISO8601:logtime}%{SPACE}\[%{DATA:thread_name}\]%{SPACE}%{LOGLEVEL:level}%{SPACE}%{GREEDYDATA:log_message}$(?<stacktrace>(?m:.*))?" }
        }
    }
    else if "httpdaccesslog" in [tags] {
        mutate {
        update => {
            "tags" => "httpdaccesslog"
            "event" => "%{[event][original]}"
            "host" => "%{[host][hostname]}"
        }
        remove_field => ["log","ecs","input","fields","agent" ,"os"]
        }
        grok {
        match => { "message" => "%{IPORHOST:clientip} %{HTTPDUSER:ident} %{USER:auth} \[%{HTTPDATE:timestamp}\] \"(?:%{WORD:verb} %{NOTSPACE:request}(?: HTTP/%{NUMBER:httpversion})?|%{DATA:rawrequest})\" %{NUMBER:response} (?:%{NUMBER:bytes}|-)" }
        }
    }
    else if "httpderrorlog" in [tags] {
        mutate {
        update => {
            "tags" => "httpderrorlog"
            "event" => "%{[event][original]}"
            "host" => "%{[host][hostname]}"
        }
        remove_field => ["log","ecs","input","fields","agent" ,"os"]
        }
        grok {
        match => { "message" => "\[%{HTTPDERROR_DATE:timestamp}\] \[(%{WORD:module})?:%{LOGLEVEL:loglevel}\] \[pid %{POSINT:pid}(:tid %{NUMBER:tid})?\]( \(%{POSINT:proxy_errorcode}\)%{DATA:proxy_message}:)?( \[client %{IPORHOST:clientip}:%{POSINT:clientport}\])?( %{DATA:errorcode}:)? %{GREEDYDATA:log_message}" }
        }
    }
    else if "redis" in [tags] {
        mutate {
        update => {
            "tags" => "redis"
            "event" => "%{[event][original]}"
            "host" => "%{[host][hostname]}"
        }
        remove_field => ["log","ecs","input","fields","agent" ,"os"]
        }
        grok {
        match => { "message" => "%{POSINT:redis_pid}:[A-Z] (?<timestamp>%{MONTHDAY} %{MONTH} %{YEAR} %{TIME}) [#*] %{GREEDYDATA:log_message}" }
        }
    }
    }

    output {
        if "storm" in [fields] {
            solr_http {
                solr_url => "http://10.1.80.178:8878/solr/storm-logs"
            }
        }
        else if [tags] == "kafkacontroller"{
        #    stdout { codec => rubydebug }
            solr_http {
                solr_url => "http://10.1.80.178:8878/solr/kafka-controller"
            }
        }
        else if [tags] == "kafkalogcleaner"{
        #    stdout { codec => rubydebug }
            solr_http {
                solr_url => "http://10.1.80.178:8878/solr/kafka-logcleaner"
            }
        }
        else if [tags] == "kafkaserver"{
        #    stdout { codec => rubydebug }
            solr_http {
                solr_url => "http://10.1.80.178:8878/solr/kafka-server"
            }
        }
        else if [tags] == "kafkastatechange"{
        #    stdout { codec => rubydebug }
            solr_http {
                solr_url => "http://10.1.80.178:8878/solr/kafka-statechange"
            }
        }
        else if [tags] == "kafkaranger"{
        #    stdout { codec => rubydebug }
            solr_http {
                solr_url => "http://10.1.80.178:8878/solr/kafka-ranger"
            }
        }
        else if [tags] == "solr"{
        #    stdout { codec => rubydebug }
            solr_http {
                solr_url => "http://10.1.80.178:8878/solr/solr-logs"
            }
        }
        else if [tags] == "httpdaccesslog"{
        #    stdout { codec => rubydebug }
            solr_http {
                solr_url => "http://10.1.80.178:8878/solr/httpd-accesslogs"
            }
        }
        else if [tags] == "httpderrorlog"{
        #    stdout { codec => rubydebug }
            solr_http {
                solr_url => "http://10.1.80.178:8878/solr/httpd-errorlog"
            }
        }
        else if [tags] == "redis"{
        #    stdout { codec => rubydebug }
            solr_http {
                solr_url => "http://10.1.80.178:8878/solr/redis-logs"
            }
        }
    }

    ```

3. Restart logstash dan hapus container filebeat lalu buat ulang 

    ```
    docker restart logstash-medan

    docker stop filebeat-medan && docker rm filebeat-medan

    docker run -d \
    --name filebeat-kafka-httpd-redis-solr \
    --hostname $(hostname) \
    -v $(pwd)/filebbeat.yml:/usr/share/filebeat/filebeat.yml \
    -v /var/log/storm/workers-artifacts:/var/log/storm/ workers-artifacts \
    -v /var/log/kafka:/var/log/kafka \
    -v /home/medan/medan2/services/redis:/home/medan/medan2/services/redis \
    -v /hadoop/lampp/httpd-2.4/logs:/hadoop/lampp/httpd-2.4/logs \
    -v /var/log/solr:/var/log/solr \
    docker.elastic.co/beats/filebeat:8.3.1

    ```

4. Cek ke solr dashboard lalu buat collection per log tunggu sampai data bisa dibaca

5. Akses banana dashboard lalu buat time-series dashboard isikan Collection Name dengan collection yang akan dimonitor ganti time field menjadi _timestamp lalu klik create 