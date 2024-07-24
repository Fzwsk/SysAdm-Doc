# Pattern multiple grok filter for Logstash

## Environment
- Logstash
- Filebeat
- Solr
- Banana

## Pattern Storm
```
^%{TIMESTAMP_ISO8601:time} %{JAVACLASS:class} %{GREEDYDATA:method} \[%{LOGLEVEL:loglevel}\]%{DATA:message}({({[^}]+},?\s*)*})?\s*$(?<stacktrace>(?m:.*))?
```

Multiline filter
```
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
```


## Pattern Solr

```
^%{TIMESTAMP_ISO8601:logtime}%{SPACE}\[%{DATA:thread_name}\]%{SPACE}%{LOGLEVEL:level}%{SPACE}%{GREEDYDATA:log_message}$(?<stacktrace>(?m:.*))?
```

Multiline filter
```
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
```

## Pattern Kafka
### pattern kafka-controller
```
^\[%{TIMESTAMP_ISO8601:logtime}\]%{SPACE}%{LOGLEVEL:level}%{SPACE}\[(?<logger>[^\]]+)\]%{SPACE}%{GREEDYDATA:log_message}
```

Multiline filter
```
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
```

### pattern kafka-server
```
^\[%{TIMESTAMP_ISO8601:logtime}\]%{SPACE}%{LOGLEVEL:level}%{SPACE}%{GREEDYDATA:log_message}
```

Multiline filter
```
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
```

### pattern kafka-ranger
```
%{TIMESTAMP_ISO8601:logtime}%{SPACE}%{LOGLEVEL:level}%{SPACE}\[(?<logger>[^\]]+)\]%{SPACE}%{GREEDYDATA:log_message}$(?<stacktrace>(?m:.*))?
```

Multiline filter
```
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
```

### pattern kafka-logcleaner
```
^\[%{TIMESTAMP_ISO8601:logtime}\]%{SPACE}%{LOGLEVEL:level}%{SPACE}%{GREEDYDATA:log_message}
```

Multiline filter
```
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
```

### pattern kafka-statechange
```
^\[%{TIMESTAMP_ISO8601:logtime}\]%{SPACE}%{LOGLEVEL:level}%{SPACE}\[(?<logger>[^\]]+)\]%{SPACE}%{GREEDYDATA:log_message}
```


filebeat yml config
```
- type: log
  enabled: true
  paths:
    - "/var/log/kafka/state-change.log"
  tags: [kafkastatechange]
```

## Pattern Httpd
### pattern access_log httpd
```
%{IPORHOST:clientip} %{HTTPDUSER:ident} %{USER:auth} \[%{HTTPDATE:timestamp}\] \"(?:%{WORD:verb} %{NOTSPACE:request}(?: HTTP/%{NUMBER:httpversion})?|%{DATA:rawrequest})\" %{NUMBER:response} (?:%{NUMBER:bytes}|-)
```

filebeat yml config
```
- type: log
  enabled: true
  paths:
    - "/hadoop/lampp/httpd-2.4/logs/access_log"
  tags: [httpdaccesslog]
```

### pattern error_log httpd
```
\[%{HTTPDERROR_DATE:timestamp}\] \[(%{WORD:module})?:%{LOGLEVEL:loglevel}\] \[pid %{POSINT:pid}(:tid %{NUMBER:tid})?\]( \(%{POSINT:proxy_errorcode}\)%{DATA:proxy_message}:)?( \[client %{IPORHOST:clientip}:%{POSINT:clientport}\])?( %{DATA:errorcode}:)? %{GREEDYDATA:log_message}
```

Multiline filter
```
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
```

## Pattern Redis

```
%{POSINT:redis_pid}:[A-Z] (?<timestamp>%{MONTHDAY} %{MONTH} %{YEAR} %{TIME}) [#*] %{GREEDYDATA:log_message}
```

filebeat.yml config
```
- type: log
  enabled: true
  paths:
    - "/home/medan/medan2/services/redis/redis-cluster-*.log"
  tags: [redis]
```


## Example full config logstash.conf
```
input {
   beats {
    port => 5044
  }
}

filter {
  if "storm" in [tags] {
    mutate {
      update => {
        "tags" => "storm"
        "event" => "%{[event][original]}"
        "host" => "%{[host][hostname]}"
      }
      remove_field => ["log","ecs","input","fields","agent" ,"os"]
    }
    grok {
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
    if [tags] == "storm"{
        solr_http {
            solr_url => "http://10.1.80.178:8878/solr/storm-logs"
        }
    }
    else if [tags] == "kafkacontroller"{
         solr_http {
             solr_url => "http://10.1.80.178:8878/solr/kafka-controller"
         }
    }
    else if [tags] == "kafkalogcleaner"{
         solr_http {
             solr_url => "http://10.1.80.178:8878/solr/kafka-logcleaner"
         }
    }
    else if [tags] == "kafkaserver"{
         solr_http {
             solr_url => "http://10.1.80.178:8878/solr/kafka-server"
         }
    }
    else if [tags] == "kafkastatechange"{
         solr_http {
             solr_url => "http://10.1.80.178:8878/solr/kafka-statechange"
         }
    }
    else if [tags] == "kafkaranger"{
         solr_http {
             solr_url => "http://10.1.80.178:8878/solr/kafka-ranger"
         }
    }
    else if [tags] == "solr"{
         solr_http {
             solr_url => "http://10.1.80.178:8878/solr/solr-logs"
         }
    }
    else if [tags] == "httpdaccesslog"{
         solr_http {
             solr_url => "http://10.1.80.178:8878/solr/httpd-accesslogs"
         }
    }
    else if [tags] == "httpderrorlog"{
         solr_http {
             solr_url => "http://10.1.80.178:8878/solr/httpd-errorlog"
         }
    }
    else if [tags] == "redis"{
         solr_http {
             solr_url => "http://10.1.80.178:8878/solr/redis-logs"
         }
    }
}
```
