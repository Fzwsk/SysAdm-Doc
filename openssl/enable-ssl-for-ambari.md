# Enable SSL for Ambari

# Environment

- Cloudera 3.x 

- Rocky 8.8

- Kerberos

# Step

# Setup SSL for Ambari

1. Create dir ssl untuk wadah dari single-signed-certificate yang nantinya akan digunakan oleh SSL

```
mkdir ssl

cd ssl/
```

2. Sebelum Create certificate sementara untuk user (root) di node Ambari-server, pastikan Ambari-server dalam kondisi stop

command create single-signed-certificate

```
wserver=$(hostname)
echo $wserver
openssl genrsa -out ${wserver}.key 2048
openssl req -new -key ${wserver}.key -out ${wserver}.csr
openssl x509 -req -days 365 -in $wserver.csr -signkey $wserver.key -out $wserver.crt
```
Note : pada saat request key & csr untuk cluster 

pada saat running `openssl req -new -key ${wserver}.key -out ${wserver}.csr` isikan seperti berikut ;

```
C=Country name
ST=State or Province
L=Locality
O=Organizational Name
OU=Organizational Unit
CN=Common Name
```

`subject=C=ID/ST=Jakarta/L=Jakarta/O=<your-organisation>/OU=<your-organisation-unit></CN=<hostname-Ambari-server>`



3. Enable HTTPS for Ambari-server

```
ambari-server setup security
```

step ketika config security untuk ambari-server

```
Pilih 1 untuk Enable HTTPS for Ambari server.
Pilih y pada Do you want to configure HTTPS ?
Pilih port yang ingin digunakan untuk SSL. masukkan Nomor port default https adalah 8443
Berikan path lengkap ke file sertifikat Anda ($ wserver.crt dari atas) dan file 
private key ($ wserver.key dari atas)
Berikan kata sandi untuk private key.
```
4. Restart ambari-server jika sudah enable https for ambari

```
ambari-server restart
```

5. Enable SSL for Cloudera Component

note : pastikan waktu running di node yang ada Ambari-server

- Create keystore untuk setiap node

note : untuk pass dari keypass & storepass disarankan harus sama <your-organisation>/OU=<your-organisation-unit

```
keytool -genkey -alias <hostname> -keyalg rsa -keysize 4096 -dname "CN=<hostname>,OU=<your-organisation-unit>,O=<your-organisation>,L=Jakarta,ST=Jakarta,C=ID" -keypass <password> -keystore node-01-keystore.jks -storepass <password>
```
contoh :

```
keytool -genkey -alias yava31el8-retest01.labs247.com -keyalg rsa -keysize 4096 -dname "CN=yava31el8-retest01.labs247.com,OU=SDO,O=Labs247,L=Jakarta,ST=Jakarta,C=ID" -keypass Geheim247! -keystore node-01-keystore.jks -storepass Geheim247!

keytool -genkey -alias yava31el8-retest02.labs247.com -keyalg rsa -keysize 4096 -dname "CN=yava31el8-retest02.labs247.com,OU=SDO,O=Labs247,L=Jakarta,ST=Jakarta,C=ID" -keypass Geheim247! -keystore node-02-keystore.jks -storepass Geheim247!


keytool -genkey -alias yava31el8-retest03.labs247.com -keyalg rsa -keysize 4096 -dname "CN=yava31el8-retest03.labs247.com,OU=SDO,O=Labs247,L=Jakarta,ST=Jakarta,C=ID" -keypass Geheim247! -keystore node-03-keystore.jks -storepass Geheim247!
```

- Eksport certificate dengan menggunakan keystore, lakukan di setiap node

note : untuk pass dari keypass & storepass disarankan harus sama

```
keytool -export -alias <hostname> -keystore node-01-keystore.jks -rfc -file node-1.crt -storepass <password>
```
contoh :

```
keytool -export -alias yava31el8-retest01.labs247.com -keystore node-01-keystore.jks -rfc -file node-1.crt -storepass Geheim247!


keytool -export -alias yava31el8-retest02.labs247.com -keystore node-02-keystore.jks -rfc -file node-2.crt -storepass Geheim247!


keytool -export -alias yava31el8-retest03.labs247.com -keystore node-03-keystore.jks -rfc -file node-3.crt -storepass Geheim247!
```

- Create truststore berdasarkan certificate yang telah di eksport menggunakan keystore, lakukan di setiap node

note : untuk pass dari keypass & storepass disarankan harus sama

```
keytool -import -noprompt -alias <hostname> -file node-1.crt -keystore node-01-truststore.jks -storepass <password>
```
contoh :

```
keytool -import -noprompt -alias yava31el8-retest01.labs247.com -file node-1.crt -keystore node-01-truststore.jks -storepass Geheim247!

keytool -import -noprompt -alias yava31el8-retest02.labs247.com -file node-2.crt -keystore node-02-truststore.jks -storepass Geheim247!

keytool -import -noprompt -alias yava31el8-retest03.labs247.com -file node-3.crt -keystore node-03-truststore.jks -storepass Geheim247!
```

- Create 1 truststore untuk menampung semua certificate yang telah di eksport, lakukan untuk setiap node

note : untuk pass dari keypass & storepass disarankan harus sama

```
keytool -import -noprompt -alias <hostname> -file node-1.crt -keystore truststore.jks -storepass <password>
```
contoh :

```
keytool -import -noprompt -alias yava31el8-retest01.labs247.com -file node-1.crt -keystore truststore.jks -storepass Geheim247!

keytool -import -noprompt -alias yava31el8-retest02.labs247.com -file node-2.crt -keystore truststore.jks -storepass Geheim247!

keytool -import -noprompt -alias yava31el8-retest03.labs247.com -file node-3.crt -keystore truststore.jks -storepass Geheim247!
```

6. Import crt yang dari ambari-server ke file `truststore.jks`

```
keytool -import -file <path-file-crt-ambari-server> -alias ambari-server -keystore <path-file-truststore.jks>
```

contoh :

```
keytool -import -file /root/ssl/yava31el8-retest01.labs247.com.crt -alias ambari-server -keystore /etc/security/serverKeys/truststore.jks
```

cek list certificate dari file `truststore.jks`, untuk mengetahui semua certificate telah diimport

```
keytool -list -keystore <path-file-truststore.jks>
```

7. Create directory untuk menyimpan keystore & truststore

```
mkdir -p /etc/security/serverKeys
```

8. Copy keystore & truststore yang telah dibuat untuk setiap host, copy sesuai dengan tempat dari alias hostname masing-masing

```
scp node-01-keystore.jks root@<hostname>:/etc/security/serverKeys/host_keystore.jks
scp node-01-truststore.jks root@<hostname>:/etc/security/serverKeys/host_truststore.jks
scp truststore.jks root@<hostname>:/etc/security/serverKeys/truststore.jks
```

contoh :

```
scp node-01-keystore.jks root@yava31el8-retest01.labs247.com:/etc/security/serverKeys/host_keystore.jks
scp node-01-truststore.jks root@yava31el8-retest01.labs247.com:/etc/security/serverKeys/host_truststore.jks
scp truststore.jks root@yava31el8-retest01.labs247.com:/etc/security/serverKeys/truststore.jks


scp node-02-keystore.jks root@yava31el8-retest02.labs247.com:/etc/security/serverKeys/host_keystore.jks
scp node-02-truststore.jks root@yava31el8-retest02.labs247.com:/etc/security/serverKeys/host_truststore.jks
scp truststore.jks root@yava31el8-retest02labs247.com:/etc/security/serverKeys/truststore.jks


scp node-03-keystore.jks root@yava31el8-retest03.labs247.com:/etc/security/serverKeys/host_keystore.jks
scp node-03-truststore.jks root@yava31el8-retest03.labs247.com:/etc/security/serverKeys/host_truststore.jks
scp truststore.jks root@yava31el8-retest03.labs247.com:/etc/security/serverKeys/truststore.jks
```

9. Setup truststore for Ambari

```
ambari-server setup-security
```
Berikut step untuk setup trustore di ambari-server

```
[root@qcupgradeyavam01 ~]# ambari-server setup-security 
Using python /usr/bin/python Security setup options...
Choose one of the following options:
[1] Enable HTTPS for Ambari server.
[2] Encrypt passwords stored in ambari.properties file. [3] Setup Ambari kerberos JAAS configuration.
[4] Setup truststore.
[5] Import certificate to truststore.
Enter choice, (1-5): 4
Do you want to configure a truststore [y/n] (y)? y TrustStore type [jks/jceks/pkcs12] (jks):jks
Path to TrustStore file :/etc/security/serverKeys/truststore.jks Password for TrustStore:
Re-enter password:
Ambari Server 'setup-security' completed successfully.
```
note : password untuk truststore pastikan sama seperti ketika create keystore & truststore di node Ambari

10. Import crt dari masing-masing host dan crt dari ambari server ke cacerts yang dari JAVA_HOME, pastikan ketika import dilakukan di setiap node

- import crt dari ambari-server ke cacerts yang dari JAVA_HOME

```
keytool -import -file /root/ssl/yava31el8-retest01.labs247.com.crt -alias ambari-server -keystore $JAVA_HOME/jre/lib/security/cacerts —storepass changeit
```

- import crt dari masing-masing host ke cacerts yang dari JAVA_HOME

```
keytool -import -file /etc/security/serverKeys/node-1.crt. -alias yava31el8-retest01.labs247.com -keystore $JAVA_HOME/jre/lib/security/cacerts —storepass changeit

keytool -import -file /etc/security/serverKeys/node-2.crt. -alias yava31el8-retest02.labs247.com -keystore $JAVA_HOME/jre/lib/security/cacerts —storepass changeit

keytool -import -file /etc/security/serverKeys/node-3.crt. -alias yava31el8-retest03.labs247.com -keystore $JAVA_HOME/jre/lib/security/cacerts —storepass changeit
```

- cek list certificate dari file `cacerts` yang dari JAVA_HOME, untuk mengetahui semua certificate telah diimport

```
keytool -list -keystore $JAVA_HOME/jre/lib/security/cacerts —storepass changeit
```

11. `Import certificate to truststore` melalui node Ambari-server

```
ambari-server setup-security
```

12. Config SSL for HDFS

```
[ssl-server]

ssl.server.keystore.location=/etc/security/serverKeys/host_keystore.jks
ssl.server.truststore.location=/etc/security/serverKeys/host_truststore.jks
ssl.server.keystore.keypassword=<key pass>
ssl.server.keystore.password=<store pass>
ssl.server.keystore.type=jks
ssl.server.truststore.password=<store pass>
ssl.server.truststore.reload.interval=10000
ssl.server.truststore.type=jks

[ssl-client]

ssl.client.keystore.location=/etc/security/serverKeys/host_keystore.jks
ssl.client.keystore.password=<store pass>
ssl.client.keystore.type=jks
ssl.client.truststore.location=/etc/security/serverKeys/truststore.jks
ssl.client.truststore.password=<store pass>
ssl.client.truststore.reload.interval=10000
ssl.client.truststore.type=jks

[hdfs-site]

hdfs.http.policy=HTTPS_ONLY
```
13. Config SSL YARN

```
[yarn-site]
yarn.http.policy=HTTPS_ONLY
yarn.log.server.url=https://<hostname>:19889/jobhistory/logs
yarn.log.server.web-service.url=https://<hostname>:8190/ws/v1/applicationhistory
yarn.nodemanager.webapp.https.address=0.0.0.0:8044
```

14. Config SSL Mapreduce

```
[mapred-site]
mapreduce.jobhistory.http.policy=HTTPS_ONLY
mapreduce.jobhistory.webapp.https.address=https://<hostname>:19889
mapreduce.shuffle.ssl.enabled=true
mapreduce.ssl.enabled=true
```

15. Config SSL Tez

```
[tez-site]
tez.runtime.shuffle.keep-alive.enabled=true
tez.runtime.shuffle.ssl.enable=true
```

16. Config SSL Spark

copy hanya di node spark-server

```
cp /etc/security/serverKeys/host_keystore.jks /etc/security/serverKeys/spark_keystore.jks
```

```
[spark-defaults-conf]
spark.authenticate=true
spark.authenticate.enableSaslEncryption=true
spark.ssl.enabled=true
spark.ssl.keyPassword=Geheim247!
spark.ssl.keyStore=/etc/security/serverKeys/spark_keystore.jks
spark.ssl.keyStorePassword=Geheim247!
spark.ssl.protocol=TLS
spark.ssl.trustStore=/etc/security/serverKeys/truststore.jks
spark.ssl.trustStorePassword=Geheim247!
spark.ui.https.enabled=true


[livy-conf]
livy.keystore=/etc/security/serverKeys/spark_keystore.jks
livy.keystore.password=<store password>
livy.key-password=<key password>


[yarn-site]
spark.authenticate=true
```
17. Config SSL Hive

di copy di all-nodes untuk keystore nya, pastikan waktu copy harus masuk ke node tsb, tidak boleh langsung di sebar krn disitu ada CN nya sendiri"

```
cp /etc/security/serverKeys/host_keystore.jks /etc/security/serverKeys/hive_keystore.jks
```

```
[hive-site.xml]
hive.server2.use.SSL=true
hive.server2.keystore.path=/etc/security/serverKeys/hive_keystore.jks
hive.server2.keystore.password=<password>
```
Akses Hive dengan jdbc url :

```
jdbc:hive2://<hostname>:<port>/<database>;ssl=true;sslTrustStore=/etc/security/serverKeys/truststore.jks;trustStorePassword=<password>;principal=hive/_HOST@REALM
```
contoh :

```
beeline -u 'jdbc:hive2://yava31el8-retest02.labs247.com:10000/default;transportMode=binary;ssl=true;sslTrustStore=/etc/security/serverKeys/truststore.jks;trustStorePassword=Geheim247!;principal=hive/_HOST@LABS247.COM'
```
18. Config SSL Hbase

```
[hbase-site]
hadoop.ssl.enabled=true
hbase.http.policy=HTTPS_ONLY
hbase.ssl.enabled=true
ssl.server.keystore.keypassword=Geheim247!
ssl.server.keystore.location=/etc/security/serverKeys/host_keystore.jks
```
19. Config SSL Infra-solr

lakukan copy di masing" node infra-solr, pastikan waktu copy harus masuk ke node tsb, tidak boleh langsung di sebar krn disitu ada CN nya sendiri"

```
cp /etc/security/serverKeys/host_keystore.jks /etc/security/serverKeys/infra_keystore.jks
````

```
[infra-solr-env]
infra_solr_ssl_enabled=true
infra_solr_keystore_location=/etc/security/serverKeys/infra_keystore.jks
infra_solr_keystore_type=jks
infra_solr_keystore_password=<password>
infra_solr_truststore_location=/etc/security/serverKeys/infra_keystore.jks
infra_solr_truststore_type=jks
infra_solr_truststore_password=<password>
```
20. Config SSL Kafka

lakukan copy di node kafka saja, pastikan waktu copy harus masuk ke node tsb, tidak boleh langsung di sebar krn disitu ada CN nya sendiri"

```
cp /etc/security/serverKeys/host_keystore.jks /etc/security/serverKeys/kafka_keystore.jks

cp /etc/security/serverKeys/host_truststore.jks /etc/security/serverKeys/kafka_keystore.jks
```


```
[kafka-broker]

ssl.keystore.location=/etc/security/serverKeys/kafka_keystore.jks
ssl.keystore.password=Geheim247!
ssl.key.password=Geheim247!
ssl.truststore.location=/etc/security/serverKeys/kafka_truststore.jks
ssl.truststore.password=Geheim247!
ssl.enabled.protocols=TLSv1.2,TLSv1.1,TLSv1
sasl.kerberos.service.name=kafka
security.inter.broker.protocol=SASL_SSL
listeners=SASL_SSL://localhost:6667
```

21. Config SSL Atlas

copy hanya di node nya atlas saja

```
cp /etc/security/serverKeys/host_keystore.jks /etc/security/serverKeys/atlas_keystore.jks
```

Membuat file jceks pada node tempat atlas terinstall dan di /home/atlas/ running command

running command

```
/usr/yava/current/atlas-server/bin/cputil.py
```

jika muncul confirm generate seperti ini

```
Please enter the full path to the credential
provider:jceks://file/home/atlas/secure.jceks 
Please enter the password value for keystore.password:<keystore pass>
Please enter the password value for keystore.password again:<keystore pass>
Please enter the password value for truststore.password:<truststore pass>
Please enter the password value for truststore.password again:<trustore pass>
Please enter the password value for password:<key pass>
Please enter the password value for password again:<keypass>
```

Note : password keystore dan truststore sama dengan password di jks.

```
[atlas-application]

keystore.file=/etc/security/serverKeys/atlas_keystore.jks
truststore.file=/etc/security/serverKeys/truststore.jks
client.auth.enabled=false
cert.stores.credential.provider.path=jceks://file//home/a
tlas/secure.jceks
atlas.ssl.exclude.cipher.suites=.*NULL.*, .*RC4.*, .*MD5.*, .*DES.*, .*DSS.*

[application-properties]

atlas.enableTLS=true
```

22. Config SSL Ranger

copy hanya di node nya ranger aja

```
cp /etc/security/serverKeys/host_keystore.jks /etc/security/serverKeys/ranger_keystore.jks
```

```
[ranger settings]

External url=https://yava31el8-retest02.labs247.com:6182 
HTTP Enabled=false  
 

[ranger-admin-site]

ranger.https.attrib.keystore.file=/etc/security/serverKeys/ranger_keystore.jks 
ranger.service.https.attrib.keystore.pass=<keystore password>  
ranger.service.https.attrib.clientAuth=false 
ranger.service.https.attrib.keystore.keyalias=<hostname/alias> 
ranger.service.https.attrib.ssl.enabled=true 

ranger.service.https.attrib.client.auth=false -> add di custom ranger-admin-site 

ranger.service.https.attrib.keystore.file=/etc/security/serverKeys/ranger_keystore.jks 


[ranger-tagsync-policymgr-ssl]

xasecure.policymgr.clientssl.keystore=/etc/security/serverKeys/ranger_keystore.jks 
xasecure.policymgr.clientssl.keystore.password=<keystore password> 
xasecure.policymgr.clientssl.truststore=/etc/security/serverKeys/truststore.jks 
xasecure.policymgr.clientssl.truststore.password=<keystore password> 
```

23. Config SSL Zeppelin

copy hanya di node zeppelin

```
cp /etc/security/serverKeys/host_keystore.jks /etc/security/serverKeys/zeppelin_keystore.jks
```

```
[zeppelin-site]

zeppelin.ssl=true
zeppelin.ssl.keystore.path=/etc/security/serverKeys/zeppelin_keystore.jks
zeppelin.ssl.truststore.path=/etc/security/serverKeys/truststore.jks
zeppelin.ssl.key.manager.password=<pass>
zeppelin.ssl.keystore.password=<pass>
zeppelin.ssl.truststore.password=<pass>
```

24. Config SSL Druid

create keystore baru di masing" node druid-server & druid-client, pastikan waktu create harus masuk ke node tsb, tidak boleh langsung di sebar krn disitu ada CN nya sendiri" atau identitas cert nya

```
keytool -genkey -alias yava31el8-retest01.labs247.com -keyalg rsa -keysize 4096 -dname "CN=yava31el8-retest01.labs247.com,OU=SDO,O=Labs247,L=Jakarta,ST=Jakarta,C=ID" -keypass Geheim247! -keystore druid-keystore.jks -storepass Geheim247!

keytool -genkey -alias yava31el8-retest02.labs247.com -keyalg rsa -keysize 4096 -dname "CN=yava31el8-retest02.labs247.com,OU=SDO,O=Labs247,L=Jakarta,ST=Jakarta,C=ID" -keypass Geheim247! -keystore druid-keystore.jks -storepass Geheim247!

keytool -genkey -alias yava31el8-retest03.labs247.com -keyalg rsa -keysize 4096 -dname "CN=yava31el8-retest03.labs247.com,OU=SDO,O=Labs247,L=Jakarta,ST=Jakarta,C=ID" -keypass Geheim247! -keystore druid-keystore.jks -storepass Geheim247!
```


```
[druid-common]

druid.client.https.certAlias=<cert-alias/hostname>
druid.client.https.keyManagerPassword=<key pass>
druid.client.https.keyStorePassword=<key pass>
druid.client.https.keyStorePath=/etc/security/serverKeys/druid_keystore.jks
druid.client.https.keyStoreType=jks
druid.client.https.protocol=TLSv1.2
druid.client.https.trustStorePassword=<trust pass>
druid.client.https.trustStorePath=/etc/security/serverKeys/truststore.jks
druid.client.https.validateHostnames=false
druid.enablePlaintextPort=false
druid.enableTlsPort=true
druid.server.https.certAlias=<cert-alias/hostname>
druid.server.https.keyStorePassword=<key pass>
druid.server.https.keyStorePath=/etc/security/serverKeys/druid_keystore.jks
druid.server.https.keyStoreType=jks

[druid router]

druid.tlsPort=9188


change port

druid-router=9188
druid-middlemanager=9291
druid-coordinator=8281
druid-overlord=8290
druid-historical=8283
druid-broker=8282
```

25. Config SSL Oozie

Copy di node oozie saja

```
cp /etc/security/serverKeys/host_keystore.jks /etc/security/serverKeys/oozie_keystore.jks
```

```
[oozie-site]

oozie.https.enabled=true
oozie.https.keystore.file=/etc/security/serverKeys/oozie_keystore.jks
oozie.https.keystore.pass=<key pass>
oozie.https.truststore.file=/etc/security/serverKeys/oozie_keystore.jks
oozie.https.truststore.pass=<trust pass>
oozie.base.url=https://<hostname>:11443
```