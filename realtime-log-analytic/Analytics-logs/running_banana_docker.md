# Running banana on docker

## environment
- docker
- centos 7 

## step 

1. pull image banana 
    ```
    docker pull aaadel/banana
    ```
2. buat file config.json 

    ```
    {
    "serverPort": 9901,
    "solrUrl": "http://192.168.3.31:8983",
    "basicAuth": false,
    "username": "",
    "password": "",
    "couchbaseUrl": "http://couchbase:8092"
    }
    ```
3. lalu start banana
    ```
    docker run -d --name banana -p 9901:9901 -v /path/to/config.json:/opt/banana/config.json aaadel/banana
    ```
4. check container berjalan atau tidak dan baca log nya
    ```
    docker ps 
    docker logs -f banana
    ```
5. apabila semua aman buka http://192.168.3.31:9901/#/dashboard
