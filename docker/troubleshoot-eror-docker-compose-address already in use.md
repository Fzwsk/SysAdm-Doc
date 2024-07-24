# Troubleshoot eror when running docker-compose up --build then find eror case address already in use

# Environment

- Centos 7

- Yiflow 2.0.0

# Issue

  ```
  ERROR: for yiflow247_web_1  Cannot start service web: driver failed programming external connectivity on endpoint yiflow247_web_1 (52f8413990f34715753ad74c20c48537c88b5538b7eeed405efc7f84ef5be065): Error starting userland proxy: listen tcp4 0.0.0.0:6001: bind: address already in use

  ERROR: for web  Cannot start service web: driver failed programming external connectivity on endpoint yiflow247_web_1 (52f8413990f34715753ad74c20c48537c88b5538b7eeed405efc7f84ef5be065): Error starting userland proxy: listen tcp4 0.0.0.0:6001: bind: address already in use
  ERROR: Encountered errors while bringing up the project.
  ```
# Solve

1. Cek port yang tertera di eror untuk,lalu lihat proses mana yang sedang berjalan atau port tersebut digunakan oleh container mana,jika sudah maka dapat diketahui bahwa port tsb dipakai oleh container siapa

   ```
   netstat -nap | grep 6001
   ```
2. kemudian cek pid dari port tsb, untuk mengetahui arah dari port tsb ke container mana
  
   ```
   ps -ef | grep 1946
   ```
3. selanjutnya ubah port yang sebelumnya sudah digunakan,ke port yang lain,untuk mengubahnya masuk ke /imeri/yiflow247/docker-compose.yml

```
   version: "3.3"
services:
  web:
    image: processmaker/pm4-core:${PM_VERSION}
    ports:
      - ${PM_APP_PORT}:80
      - ${PM_BROADCASTER_PORT}:6002:6001
    environment:
      - PM_APP_URL
      - PM_APP_PORT
      - PM_BROADCASTER_PORT
    volumes:
      - ${PM_DOCKER_SOCK}:/var/run/docker.sock
      - storage:/code/pm4/storage
      - ./source_code/yiflow/:/code/pm4
    links:
      - redis
      - mysql
    depends_on:
      - mysql
      - redis
  redis:
    image: redis
  mysql:
    image: mysql:5.7
    restart: always
    environment:
      MYSQL_ROOT_PASSWORD: password
      MYSQL_DATABASE: processmaker
      MYSQL_USER: pm
      MYSQL_PASSWORD: pass
    volumes:
      - database:/var/lib/mysql
volumes:
  database:
  storage:
```
4. jika sudah running docker-compose up --build ulang dari repository git nya

   ```
   docker-compose up --build
   ```


