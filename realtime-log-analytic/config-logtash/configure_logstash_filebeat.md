# Configure logstash filebeat connect to solr

## Environment

- centos 7
- logstash
- filebeat


## Installing logstash

First of all, install logstash using this command :

Download and install public signing key:

```
sudo rpm --import https://artifacts.elastic.co/GPG-KEY-elasticsearch
```
Add these config in the new file at */etc/yum.repos.d/* directory with file named *logstash.repo*

```
[logstash-8.x]
name=Elastic repository for 8.x packages
baseurl=https://artifacts.elastic.co/packages/8.x/yum
gpgcheck=1
gpgkey=https://artifacts.elastic.co/GPG-KEY-elasticsearch
enabled=1
autorefresh=1
type=rpm-md
```
After adding repo, now install it using these command

```
sudo yum install logstash
```

## Installing Filebeat
First of all, import publick signing key using command :

```
sudo rpm --import https://packages.elastic.co/GPG-KEY-elasticsearch
```

Create a file named *elastic.repo* and add these config below

```
[elastic-8.x]
name=Elastic repository for 8.x packages
baseurl=https://artifacts.elastic.co/packages/8.x/yum
gpgcheck=1
gpgkey=https://artifacts.elastic.co/GPG-KEY-elasticsearch
enabled=1
autorefresh=1
type=rpm-md
```

Finally after adding new repo, install filebeat using these command

```
sudo yum install filebeat
```

## Setting Up Logstash

Go to */usr/share/logstash* and install plugin solr_http with these command :

```
bin/logstash-plugin install logstash-output-solr_http
```
after done installing plugin search file named solr_http.rb with command

```
find / -type f -name solr_http.rb
```

and then apply patch like example below

```
   config :document_id, :validate => :string, :default => nil
 
+  # Document transformation XSL
+  config :tr, :validate => :string, :default => nil
+
   public
   def register
     require "rsolr"
-    @solr = RSolr.connect :url => @solr_url
+    @solr = RSolr.connect :url => @solr_url, update_format: :xml
     buffer_initialize(
       :max_items => @flush_size,
       :max_interval => @idle_flush_time,
@@ -62,7 +65,7 @@
 
     events.each do |event|
         document = event.to_hash()
-        document["@timestamp"] = document["@timestamp"].iso8601 #make the timestamp ISO
+        document["@timestamp"] = document["@timestamp"].to_iso8601 #make the timestamp ISO
         if @document_id.nil?
           document ["id"] = UUIDTools::UUID.random_create    #add a unique ID
         else
@@ -71,7 +74,7 @@
         documents.push(document)
     end
 
-    @solr.add(documents)
+    @solr.add(documents, :add_attributes => {:commitWithin=>10000}, :params => {:tr => @tr})
     rescue Exception => e
       @logger.warn("An error occurred while indexing: #{e.message}")
   end #def flush
```
After applying some patch go to bin directory

```
cd /usr/share/logstash/bin
```
and find file named logstash-filter.conf and set the value same as below

```
input {
   beats {
    port => 5044
  }
}

filter {
  if "dummy" in [fields][type] {
    mutate {
        remove_field => [ "log","ecs","input","tags","host","fields","agent" ,"os" ]
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
  stdout { codec => rubydebug }
    solr_http {
      id => "solr_plugin_1"
      solr_url => "http://localhost:8983/solr/logs"
    }
}
```


## Setting Up Filebeat

Go to */etc/filebeat/filebeat.yml* and change some value like examples below

```
cd /etc/filebeat/filebeat.yml
```

```
###################### Filebeat Configuration Example #########################

# This file is an example configuration file highlighting only the most common
# options. The filebeat.reference.yml file from the same directory contains all the
# supported options with more comments. You can use it as a reference.
#
# You can find the full configuration reference here:
# https://www.elastic.co/guide/en/beats/filebeat/index.html

# For more available modules and options, please see the filebeat.reference.yml sample
# configuration file.

# ============================== Filebeat inputs ===============================

filebeat.inputs:

# Each - is an input. Most options can be set at the input level, so
# you can use different inputs for various configurations.
# Below are the input specific configurations.


- type: log
  enabled: true
  paths:
    - /var/dummy/dummy.log
  fields:
    type: dummy

# filestream is an input for collecting log messages from files.
- type: filestream

  # Unique ID among all inputs, an ID is required.
  id: my-filestream-id

  # Change to true to enable this input configuration.
  enabled: false

  # Paths that should be crawled and fetched. Glob based paths.
  paths:
    - /var/log/*.log
    #- c:\programdata\elasticsearch\logs\*

  # Exclude lines. A list of regular expressions to match. It drops the lines that are
  # matching any regular expression from the list.
  #exclude_lines: ['^DBG']

  # Include lines. A list of regular expressions to match. It exports the lines that are
  # matching any regular expression from the list.
  #include_lines: ['^ERR', '^WARN']

  # Exclude files. A list of regular expressions to match. Filebeat drops the files that
  # are matching any regular expression from the list. By default, no files are dropped.
  #prospector.scanner.exclude_files: ['.gz$']

  # Optional additional fields. These fields can be freely picked
  # to add additional information to the crawled log files for filtering
  #fields:
  #  level: debug
  #  review: 1

# ============================== Filebeat modules ==============================

filebeat.config.modules:
  # Glob pattern for configuration loading
  path: ${path.config}/modules.d/*.yml

  # Set to true to enable config reloading
  reload.enabled: false

  # Period on which files under path should be checked for changes
  #reload.period: 10s

# ======================= Elasticsearch template setting =======================

setup.template.settings:
  index.number_of_shards: 1
  #index.codec: best_compression
  #_source.enabled: false


# ================================== General ===================================

# The name of the shipper that publishes the network data. It can be used to group
# all the transactions sent by a single shipper in the web interface.
#name:

# The tags of the shipper are included in their own field with each
# transaction published.
#tags: ["service-X", "web-tier"]

# Optional fields that you can specify to add additional information to the
# output.
#fields:
#  env: staging

# ================================= Dashboards =================================
# These settings control loading the sample dashboards to the Kibana index. Loading
# the dashboards is disabled by default and can be enabled either by setting the
# options here or by using the `setup` command.
#setup.dashboards.enabled: false

# The URL from where to download the dashboards archive. By default this URL
# has a value which is computed based on the Beat name and version. For released
# versions, this URL points to the dashboard archive on the artifacts.elastic.co
# website.
#setup.dashboards.url:

# =================================== Kibana ===================================

# Starting with Beats version 6.0.0, the dashboards are loaded via the Kibana API.
# This requires a Kibana endpoint configuration.
setup.kibana:

  # Kibana Host
  # Scheme and port can be left out and will be set to the default (http and 5601)
  # In case you specify and additional path, the scheme is required: http://localhost:5601/path
  # IPv6 addresses should always be defined as: https://[2001:db8::1]:5601
  #host: "localhost:5601"

  # Kibana Space ID
  # ID of the Kibana Space into which the dashboards should be loaded. By default,
  # the Default Space will be used.
  #space.id:

# =============================== Elastic Cloud ================================

# These settings simplify using Filebeat with the Elastic Cloud (https://cloud.elastic.co/).

# The cloud.id setting overwrites the `output.elasticsearch.hosts` and
# `setup.kibana.host` options.
# You can find the `cloud.id` in the Elastic Cloud web UI.
#cloud.id:

# The cloud.auth setting overwrites the `output.elasticsearch.username` and
# `output.elasticsearch.password` settings. The format is `<user>:<pass>`.
#cloud.auth:

# ================================== Outputs ===================================

# Configure what output to use when sending the data collected by the beat.

# ---------------------------- Elasticsearch Output ----------------------------
#output.elasticsearch:
  # Array of hosts to connect to.
  #hosts: ["localhost:9200"]

  # Protocol - either `http` (default) or `https`.
  #protocol: "https"

  # Authentication credentials - either API key or username/password.
  #api_key: "id:api_key"
  #username: "elastic"
  #password: "changeme"

# ------------------------------ Logstash Output -------------------------------
output.logstash:
  # The Logstash hosts
  hosts: ["localhost:5044"]

  # Optional SSL. By default is off.
  # List of root certificates for HTTPS server verifications
  #ssl.certificate_authorities: ["/etc/pki/root/ca.pem"]

  # Certificate for SSL client authentication
  #ssl.certificate: "/etc/pki/client/cert.pem"

  # Client Certificate Key
  #ssl.key: "/etc/pki/client/cert.key"

# ================================= Processors =================================
processors:
  - add_host_metadata:
      when.not.contains.tags: forwarded
  - add_cloud_metadata: ~
  - add_docker_metadata: ~
  - add_kubernetes_metadata: ~

# ================================== Logging ===================================

# Sets log level. The default log level is info.
# Available log levels are: error, warning, info, debug
#logging.level: debug

# At debug level, you can selectively enable logging only for some components.
# To enable all selectors use ["*"]. Examples of other selectors are "beat",
# "publisher", "service".
#logging.selectors: ["*"]

# ============================= X-Pack Monitoring ==============================
# Filebeat can export internal metrics to a central Elasticsearch monitoring
# cluster.  This requires xpack monitoring to be enabled in Elasticsearch.  The
# reporting is disabled by default.

# Set to true to enable the monitoring reporter.
#monitoring.enabled: false

# Sets the UUID of the Elasticsearch cluster under which monitoring data for this
# Filebeat instance will appear in the Stack Monitoring UI. If output.elasticsearch
# is enabled, the UUID is derived from the Elasticsearch cluster referenced by output.elasticsearch.
#monitoring.cluster_uuid:

# Uncomment to send the metrics to Elasticsearch. Most settings from the
# Elasticsearch output are accepted here as well.
# Note that the settings should point to your Elasticsearch *monitoring* cluster.
# Any setting that is not set is automatically inherited from the Elasticsearch
# output configuration, so if you have the Elasticsearch output configured such
# that it is pointing to your Elasticsearch monitoring cluster, you can simply
# uncomment the following line.
#monitoring.elasticsearch:

# ============================== Instrumentation ===============================

# Instrumentation support for the filebeat.
#instrumentation:
    # Set to true to enable instrumentation of filebeat.
    #enabled: false

    # Environment in which filebeat is running on (eg: staging, production, etc.)
    #environment: ""

    # APM Server hosts to report instrumentation results to.
    #hosts:
    #  - http://localhost:8200

    # API Key for the APM Server(s).
    # If api_key is set then secret_token will be ignored.
    #api_key:

    # Secret token for the APM Server(s).
    #secret_token:


# ================================= Migration ==================================

# This allows to enable 6.7 migration aliases
#migration.6_to_7.enabled: true

```

After changing some value restart filebeat service using command

```
systemctl restart filebeat
```

## Making test logs 

make a new directory in */var/dummy*

```
mkdir -p /var/dummy
```

create a file named dummy_log_generator.sh

```
#!/bin/bash

while true
do 
	echo 'Dec 23 12:11:43 louis postfix/smtpd[31499]: connect from unknown[95.75.93.154]' >> /var/dummy/dummy.log
	echo "Dec 23 14:42:56 louis named[16000]: client 199.48.164.7#64817: query (cache) \'amsterdamboothuren.com/MX/IN\' denied" >> /var/dummy/dummy.log
	echo 'Dec 23 14:30:01 louis CRON[619]: (www-data) CMD (php /usr/share/cacti/site/poller.php >/dev/null 2>/var/log/cacti/poller-error.log)' >> /var/dummy/dummy.log
	echo 'Dec 22 18:28:06 louis rsyslogd: [origin software="rsyslogd" swVersion="4.2.0" x-pid="2253" x-info="http://www.rsyslog.com"] rsyslogd was HUPed, type /'lightweight'."' >> /var/dummy/dummy.log
	sleep 5
done
```

And run command to check output

```
tail -f /var/dummy/dummy.log
```

# cek output

Check output using command, and make sure solr is installed first and already have new collection :

```
tail -f /var/dummy/dummy.log
```

go to */etc/logstash/conf.d* and run command

```
/usr/share/logstash/bin/logstash -f /etc/logstash/conf.d/logstash-filter.conf
```

check in Solr UI *http://localhost:8983/logs* to make sure logs already saved to Solr