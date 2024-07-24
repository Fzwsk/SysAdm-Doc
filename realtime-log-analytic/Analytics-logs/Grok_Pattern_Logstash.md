# Grok Pattern Logstash


## Pattern Storm
```
^%{TIMESTAMP_ISO8601:time} %{JAVACLASS:class} %{GREEDYDATA:method} \[%{LOGLEVEL:loglevel}\]%{DATA:message}({({[^}]+},?\s*)*})?\s*$(?<stacktrace>(?m:.*))?
```

Multiline filter
```
^202
```


## Pattern Solr
```
^%{TIMESTAMP_ISO8601:logtime}%{SPACE}\[%{DATA:thread_name}\]%{SPACE}%{LOGLEVEL:level}%{SPACE}%{GREEDYDATA:log_message}$(?<stacktrace>(?m:.*))?
```

Multiline filter
```
^202
```

## Pattern Kafka
```
^\[%{TIMESTAMP_ISO8601:logtime}\]%{SPACE}%{LOGLEVEL:level}%{SPACE}%{GREEDYDATA:log_message}
```
Multiline filter
```
^\[202
```

## Pattern Httpd
pattern access_log httpd
```
%{IPORHOST:clientip} %{HTTPDUSER:ident} %{USER:auth} \[%{HTTPDATE:timestamp}\] "(?:%{WORD:verb} %{NOTSPACE:request}(?: HTTP/%{NUMBER:httpversion})?|%{DATA:rawrequest})" %{NUMBER:response} (?:%{NUMBER:bytes}|-)
```
pattern error_log

```
%{HTTPD20_ERRORLOG}|%{HTTPD24_ERRORLOG}
```
## Pattern Redis
```
%{POSINT:redis_pid}:[A-Z] %{MONTHDAY:day} %{MONTH:month} %{YEAR:year} %{HOUR:hour}:%{MINUTE:minute}:%{SECOND:second} [#*] %{GREEDYDATA:log_message}
```

```
(?<timestamp>%{MONTHDAY} %{MONTH} %{YEAR} %{TIME})
```