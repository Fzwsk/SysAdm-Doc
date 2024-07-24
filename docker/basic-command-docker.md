basic command docker 

1. melihat semua docker image

   ```
   docker image ls
   ```
2. download image from docker hub

   ```
   docker imagel pull isikannamaimage:isikanjenistag
   ```
3. delete image 

   ```
   docker image rm isikannamaimage:isikanjenistag
   ```
4. melihat semua docker container, entah sedang running atau stoped,maupun yang baru di created

   ```
   docker container ls -a
   ```
5. melihat semua docker yang sedang berjalan saja

   ```
   docker container ls
   docker ps -a
   ```
6. membuat container di docker dengan image yang sudah di pull dari docker registry

   -pastikan sudah mendownload image

   -container dapat di created ulang dengan image yg sama, asalkan nama dari container harus beda, berikut command untuk create nya

    ```
    docker container create --name isikannamacontainer isikannamaimage:isikanjenistag
    ```
7. menjalankan container

   -jika sudah running maka statusnya up 

    ```
    docker container start namacontainer
    ```
jika dalam running,container ada sedikit eror,maka container yg lain tidak akan berpengaruh,krn container sudah terisolasi oleh docker

8. menghentikan container

   -jika sudah stopped maka statusnya exited

   ```
   docker container stop namacontainer
   ```
9. menghapus container

   -untuk menghapus container,pastikan container yang akan dihapus sudah berstatus stopped alias exited

   ```
   docker container rm namacontainer
   ```
10. melihat log aplikasi di container kita

   -fungsinya untuk mempermudah pada saat debuging,agar lebih mudah untuk mengetahui letak erornya dimana docker container logs namacontainer

   -jika ingin mengetahui logs yg terbaru dari sebuah container maka tambahkan flag -f

    ```
    docker container logs -f namacontainer
    ```

   atau

    ```
    docker logs -f --tail 50 namacontainer
    ```
11. menggunakan command container-exec

    -command ini untuk mengeksekusi kode program yg ada di dalam container

    -bukan hanya mengeksekusi saja,tp bisa masuk ke dalam docker container melalui image yang biasanya dibuat melalui linux

     ```
     docker container exec -i -t namacontainer /bin/bash
     ```
     atau

     ```
     docker container exec -it namacontainer /bin/bash
     ```
     - /bin/bash itu artinya akan masuk ke folder /bin/bash dan akan mengeksekusi program kode yang ada didalam docker image

     -i artinya argument interaktif,agar inputnya tetap aktif

     -t artinya argument agar dapat alokasi ke terminal akses

12. Port Forwarding di docker container

    -menerapkan container port agar dari host(laptop kita/vm kita) dapat mengakses apk yg ada di dalam container docker 

    -ketika hendak menerapkan port forwarding,pastikan diterapkan pada saat membuat container baru

    -port forwarding ini berfungsi agar dari host bisa langsung masuk ke dalam container yang akan dituju,tanpa harus menerapkan container exec ke dalam container yg akan dituju

    -untuk mengetahui port sudah terhubung atau belum,maka start dulu container yg sudah di port forwarding 

    -lalu jika sudah di start,coba panggil di browser localhost:portdarihost(isikanportforwardingnya)

    -jika container apk tidak di start,maka ketika memanggil localhost melalui browser, tidak akan menampilkan isi dari apk container yg sedang dituju

    berikut command port forwarding

    ```
    docker container create --name isikannamacontainerbaru --publish isikanportdarihost(gunakan yg 8080 atau mau disamakan dg port dari image apk):isikanportdefaultdariimage isikannamaimage:isikantag
    ```

    note= jika ingin membuat container yang sama, pastikan dalam menerapkan port forwarding itu tidak sama,yang sama boleh port yang dari image,untuk port yang dari host jangan sama 

13. Menerapkan Container environment variabel

    -fungsinya untuk mengubah konfigurasi yang ada di aplikasi,tanpa harus masuk ke dalam apk containernya,cukup melalui environtmentnya saja

    -jadi container environtmer variabel ini untuk mengubah konfigurasi dari apk yg ada di containernya,tetapi mengubahnya bukan dari dalam container melainkan dari luar container

    -untuk penerapannya kali ini akan mengubah value dari user/pass dari dalam apk container

    berikut command env variabel

    ```
    docker container create --name isikannamacontainerbaru --publish isikanportdarihost(gunakan yg 8080 atau mau disamakan dg port dari image apk):isikanportdefaultdariimage --env KEY DARI USER="value" --env KEY DARI PASS="value" isikannamaimage:isikantag
    ```
    note=klo value yg dari env itu lebih dari satu kata maka pake tanda petik dua "" dan valuenya bisa diubah ketika menerapkan parameter --env key dari environtment bisa didapatkan di description image yg ada di docker hub

    -lalu jika sudah di start,coba panggil di browser localhost:portdarihost(biasanya8080)

    -jika container apk tidak di start,maka ketika memanggil localhost melalui browser, tidak akan menampilkan isi dari apk container yg sedang dituju

    - cek di dbclient untuk yang membuat container dg image database, untuk dbclientnya pake dbeaver

    - masuk ke dbeaver dan connectkan ke db yang mau dipake

    - jika sudah isikan host dengan ip dari server, lalu pastikan port sudah sesuai, untuk nama database tidak usah diisi,user/pass adalah yang sebelumnya sudah dibuat disaat menerapkan --env dengan key user dan pass

14. Mengetaui resource yang ada dari sebuah container

    -untuk mengetahui daftar resource yang digunakan oleh container itu apa saja,maka gunakan perintah 

    ```
    docker container stats
    ```
15. Membatasi penggunaan resource pada container docker

    -klo dockernya diinstall di host windows maka akan diminta untuk mengatur resource seperrti memory dan cpu,tetapi klo docker nya diinstall di linux maka akan mengikuti resource yg ada di linux tanpa harus mengaturnya

    -Nah apabila semua container dibuat di host linux,dan tidak dibatasi limit resourcenya,maka akan berpengaruh tentang perform dari container yg lain

    -untuk mengaturnya tinggal tambahkan parameter --memory dan --cpus

    --memory valuenya biasanya;

    b artinya byte, k artinya kilobyte, m artinya megabyte, g artinya gigabyte

    --cpus valuenya bisa diatur selimit mungkin misal;

    0.5 atau 1.5 jd dibacanya menggunakan setengah core atau satu setengah core

    -pastikan jika melakukan limit resource dilakukan diawal ketika hendak membuat container baru

    untuk contoh command nya seperti ini;

    ```
    docker container create --name isikannamacontainerbaru --memory 100m --cpus 1.5 --publish isikanportdarilaptop(gunakan yg 8080 atau mau disamakan dg port dari image apk):isikanportdefaultdariimage isikannamaimage:isikantag
    ```
    diatas menerapkan resource limit 100m untuk memory dan 1.5 untuk penggunaan core nya

16. Manajemen data di docker dengan menerapkan bind mounts

    -untuk melakukan sharing file atau folder dari host ke container atau sharing dari file atau folder yang dilakukan dari dalam container ke dalam file atau folder yang ada di host

    -tapi lebih disarankan bind mounts dari data yang sudah ada di dalam file atau folder dari container,dan baru di sharing ke file atau folder yang ada di host

    -untuk melakukan sharing,tinggal gunakan parameter --mount 

    -isi dari parameter mount sebagai berikut;

    -type bisa diisi dengan bind atau volume (untuk saat ini menggunakan bind)

    -source isinya letak folder milik host yang untuk menyimpan data dari container apabila sudah dilakukan sharing untuk mengetahui dimana letaknya cukup gunakan perintah pwd, sebelum pwd jangan lupa buat dir baru untuk menyimpan data yang nantinya akan di sharing dari container 

    -destination isinya letak data dimana disimpan di dalam container, untuk destination bisa diketahui letak dimana data disimpan di dalam container,dengan mencari di docker hub sesuai image yang digunakan di container dengan key "where stored data""

    berikut contoh commandnya;

    ```
    docker container create --name isikannamacontainerbaru --publish isikanportdarilaptop(gunakan yg 8080 atau mau disamakan dg port dari image apk):isikanportdefaultdariimage --mount "type=bind,source=tuliskanletakfolderdarihost,destination=isikanletakdimanadatadisimpandariimage" --env KEY DARI USER="value" --env KEY DARI PASS="value" isikannamaimage:isikantag
    ```
    note = jika data sudah di sharing ke folder dari host,dan bila container nya di stop dan didelete,maka data yg sebelumnya disharing tidak akan ikut kehapus dan jika commandnya dijalankan kembali maka akan tetap bisa ditampilkan lagi isi dari datanya

    cara untuk bind mounts :

    -buat folder di host terlebih dahulu

    ```
    mkdir namafolderbaru
    ```
    -kemudian running command seperti yang sudah dijelaskan yg diatas

    -jika sudah cek containernya

    ```
    docker ps -a
    ```
    -kemudian coba cek isi data dari value yang ada di destination pada saat melakukan mount

    ```
    docker exec -it namacontainer
    ```
    kemudian

    ```
    cd /var/lib/mysql
    ```
   lalu ll atau ls -lrth

   -jika sudah ada datanya di folder mysql, maka start pada container yang menerapkan bind mount

   -kemudian masuk ke folder yang sudah di buat di sistem host nya dan cek apakah data dari folder mysql sudah berada di folder yang barusan dibuat untuk mount ( foldernya ada di value dari source )

   -jika datanya sudah ada maka bisa dikatakan dalam bind mount nya berhasil

17. Manajemen data dengan docker volume

    -fitur ini lebih disarankan untuk digunakan ketimbang bind mounts

    -perbedaan bind mounts dengan docker volume itu klo bind mounts itu datanya disimpan di sistem host,sedangkan docker volume terdapat manajemen volume,dimana dari manajemen volume itu terdiri dari membuat volume,melihat daftar volume,dan menghapus volume

    -volume bisa diibaratkan dengan storage jadi volume ini untuk menyimpan data yang di manage oleh docker

    -saat kita membuat container,tanpa menggunakan bind mount maka ketika sudah dibuat,maka data akan disimpan di volume,dan dalam dalam penamaan volumenya secara auto-generated,jd namanya hanya berbentuk seperti id number

18. melihat daftar volume

    ```
    docker volume ls
    ```
19. membuat docker volume

    ```
    docker volume create namavolume
    ```
20. menghapus docker volume
    -untuk menghapus,pastikan docker volume sedang tidak digunakan oleh container, atau jika bingung untuk mengetahui status volumenya sedang digunakan atau tidak maka tinggal stop pada container dan delete pada container,lalu delete volumenya
    
    ```
    docker volume rm namavolumeygmaudihapus
    ```
21. Menghubungkan volume ke sebuah Container

    -volume yang sudah dibuat,bisa digunakan oleh container

    -keuntungan dari penggunaan volume itu,klo containernya dihapus,maka data akan tetap tersimpan di volume dan tidak ada perubahan untuk sisi datanya

    -cara menggunakan volume,agar dapat digunakan oleh container,berikut command nya;
 
    ```
    docker container create --name isikannamacontainerbaru --mount "type=volume,source=tuliskannamavolumeygbarudibuat,destination=isikanletakdimanadataygdariimagedisimpan" --publish isikanportdarilaptop(gunakan yg 8080 atau mau disamakan dg port dari image apk):isikanportdefaultdariimage --env KEY DARI USER="value" --env KEY DARI PASS="value" isikannamaimage:isikantag
    ```
    note = jika data sudah di sharing ke volume,dan bila container nya di stop dan didelete,maka data yg sebelumnya disharing tidak akan ikut kehapus

    dan jika commandnya dijalankan kembali dg menggunakan volume yang sama,maka akan tetap bisa ditampilkan lagi isi dari datanya

21. Backup Volume

    Dalam melakukan backup volume,terdapat dua cara yang umum digunakan,berikut beberapa penjelasannya;

    backup volume secara manual

    -untuk melakukan backup volume,pastikan volume yang akan dibackup sedang tidak digunakan oleh container,alias containernya sudah dalam status exited atau stop stop dulu container nya,apabila ingin menggunakan volumenya untuk dibackup;

    ```
    docker container stop namacontainer
    ```
    -kemudian buat folder baru di sistem host,dimana folder yg baru dibuat itu untuk tempat menyimpan backup nantinya

    ```
    mkdir namadirbaru
    ```
    -buat container baru dengan menerapkan type mount yaitu bind dan volume di parameter --mount nya

    ```
    docker container create --name isikannamacontainerbaru --mount "type=bind,source=letakdimanafolderygbarudibuat",destination=/namadirbaru" --mount "type=volume,source=isikannamavolumeygmaudibacup,destination=/data" namaimage:tag
    ```
    kemudian start container yang barusan dibuat 

    ```
    docker container start isikannamacontainerbaru
    ```
    -jika sudah maka masuk ke container-exec

    ```
    docker container exec -i -t namacontaineryangvolumenyasedangdibackup /bin/bash
    ```
    -dan cari folder /namadirbaru dan /data, jika sudah maka masuk ke folder tsb dan ls -l atau ll

    -lalu masuk ke folder /namadirbaru

    -kemudian lakukan archive data dari folder /data ke folder /namadirbaru

    untuk archive nya commandnya seperti ini;

    ```
    tar cvf /namadirbaru/namafile.tar.gz /data
    ```
    -lalu cek di folder /namadirbaru jika ada file archive namafile.tar.gz maka dalam archive data berhasil

    -kemudian stop dan delete container yang barusan digunakan untuk backup data

    ```
    docker contianer stop isikannamacontainerbaru

    docker container rm isikannamacontainerbaru
    ```

    -lalu start pada containeryangvolumenyasedangdibackup sebelumnya 

    ```
    docker container start namacontainer
    ```

    dan cek di dbeaver client

    backup volume secara langsung

    -Dengan cara ini kita hanya melakukan stop pada container yang volumenya akan dibackup,kemudian running command backup volume, dan start lagi pada container yang volumenya akan dibackup

    -kemudian untuk running command backup volume nya itu menggunakan perintah run dan diikuti parameter --rm dimana jika volume sudah di backup,maka secara otomatis akan meremove container apabila perintahnya selesai berjalan

    berikut command nya ;

    ```
    docker container stop namacontainerygmaudibackupvolumenya
    ``` 

    pastikan ketika hendak running command backup volume, volume yang digunakan oleh container sudah di stop

    untuk running command backup volume, yg dibawah itu klo sudah pernah backup volume ke dir sebelumnya ( ke letakdimanafolderygbarudibuat )

    ```
    docker container run --rm --name isikannamacontainerbarubuatbackupnya --mount "type=bind,source=letakdimanafolderygbarudibuat",destination=/namadirbaru" --mount "type=volume,source=isikannamavolumeygmaudibacup,destination=/data" namaimage:tag tar cvf /namadirbaru/namafile-lagi.tar.gz /data"  
    ```
    nah untuk yg dibawah,adalah running command backup volume untuk yang belum pernah backup volume dir yang baru buat (ke letakdimanafolderygbarudibuat) dengan cara melakukan sharing data yang ada di folder /data, dimana /data itu ada di /bin/bash 
 
    ```
    docker container run --rm --name isikannamacontainerbarubuatbackupnya --mount "type=bind,source=letakdimanafolderygbarudibuat",destination=/data" --mount "type=volume,source=isikannamavolumeygmaudibacup,destination=/data" namaimage:tag tar cvf /namadirbaru/namafile-lagi.tar.gz /data"  
    ```
    -pastikan nama file archive nya sedikit beda dengan yg sebelumnya

    -untuk yg running command backup volume,bisa dipilih sesuai dengan kegunaan dan keadaannya 

    ```
    docker container start namacontainerygmaudibackupvolumenya
    ```
22. Restore Volume

    buat volume baru untuk restore volume

    ```
    docker volume create namavolumebaru
    ```
    untuk command restore nya sama seperti backup tetapi sedikit berbeda

    ```
    docker container run --rm --name isikannamacontainerbarubuatrestorenya --mount "type=bind,source=letakdimanafolderygbarudibuat",destination=/namadirbaru" --mount "type=volume,source=isikannamavolumebaru,destination=/data" namaimage:tag bash -c "cd /data && tar xvf /namadirbaru/namafilebaru.tar.gz --strip 1"
    ```
    perintah untuk restore volume dengan cara backup file ke dalam volume, perintah ini jadi satu dengan sebelumnya 

    bash -c "cd /data && tar xvf /namadirbaru/namafilebaru.tar.gz --strip 1

    jika sudah maka buat container baru,dengan menerapkan port forwarding, bind mount dan environment variabel 

    ```
    docker container create --name isikannamacontainerbaru --publish isikanportdarihost:portdariimage --mount "type=volume,source=namavolumebaru,destination=isikanletakdimanadatadariimagedisimpan" --env KEY DARI USER="value" --env KEY DARI PASS="value" image:tag
    ```
    docker inspect namacontainer itu fungsinya untuk mengetahui informasi secara detail dari sebuah container atau image yang lain

    membuat image

    - membuat folder baru di sistem, lalu masuk ke folder tsb

    - buat dockerfile tanpa ada ekstensi seperti .sh .exe

    dockerfile itu isinya tentang perintah yang nantinya dapat dijalankan berdasarkan image dan command yang akan dibuat di file dari dockerfile

    ```
    vi dockerfile
    ```
    berikut perintah command yang ada di dockerfile;

    ARG – Instruksi ini memungkinkan Anda untuk menentukan variabel yang dapat dilewati saat build-time. Anda juga dapat menentukan nilai default.

    FROM – image dasar untuk membangun image baru. Instruksi ini harus menjadi instruksi non-comment pertama di Dockerfile.

    Satu-satunya pengecualian dari aturan ini adalah ketika Anda ingin menggunakan variabel dalam argumen FROM. Dalam hal ini FROM dapat didahului dengan satu atau lebih instruksi ARG.

    LABEL – Digunakan untuk menambahkan metadata ke image, seperti deskripsi, versi, author dll. Anda dapat menentukan lebih dari satu LABEL dan setiap instruksi LABEL adalah key-value pair.

    RUN – Perintah yang ditentukan dalam instruksi ini akan dieksekusi selama proses build. Setiap instruksi RUN membuat layer baru di atas image saat ini.

    ADD – Digunakan untuk menyalin file dan direktori dari sumber yang ditentukan ke tujuan yang ditentukan pada image docker. Sumbernya dapat berupa file atau direktori lokal atau URL. Jika sumbernya adalah arsiptar lokal , maka secara otomatis dibongkar ke dalam image Docker.

    COPY – Mirip dengan ADD tetapi sumbernya hanya berupa file atau direktori lokal.

    ENV – Instruksi ini memungkinkan Anda untuk mendefinisikan environment variable.

    CMD – Digunakan untuk menentukan perintah yang akan dieksekusi ketika Anda menjalankan sebuah container. Anda hanya dapat menggunakan satu instruksi CMD di Dockerfile Anda.

    ENTRYPOINT – Mirip dengan CMD, instruksi ini mendefinisikan perintah apa yang akan dieksekusi ketika menjalankan sebuah container.

    WORKDIR – Directive ini menetapkan direktori kerja saat ini untuk instruksi RUN, CMD, ENTRYPOINT, COPY, dan ADD berikut.

    USER – Tetapkan nama user atau UID untuk digunakan ketika menjalankan instruksi RUN, CMD, ENTRYPOINT, COPY dan ADD.

    USER – Set the username or UID to use when running any following RUN, CMD, ENTRYPOINT, COPY and ADD instructions.

    VOLUME – Memungkinkan Anda untuk memasang direktori mesin host ke container.

    EXPOSE – Digunakan untuk menentukan port tempat Container mendengarkan saat runtime.

    -jika sudah maka build agar dockerfile dapat dijalankan 

    ```
    docker build --tag namaimagerandom .
    ```
    tanda titik itu mengartikan bahwa sedang didalam folder yang menyimpan dockerfile

    nah klo tidak di folder tsb,maka tinggal sebut saja letak file dari dockerfile itu ada di folder mana

    -jika sudh maka create container dengan image yang barusan sudah dibuat

    ```
    docker run -d --name isikannamacontaineryangmaudibuat image:tag
    ```

    membuat docker-compose

    -masuk ke folder /usr/local/bin

    -lalu install docker-compose dengan perintah;
    
    ```
    curl -L "https://github.com/docker/compose/releases/download/1.23.1/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin docker-compose
    ```
    -jika sudah lakukan perintah ll untuk memastikan apakah file tsb sudah ada atau belum

    -lalu add permition nya agar file docker-compose dapat dieksekusi

    ```
    chmod +x /usr/local/bin/docker-compose
    ```
    -kemudian cek versi dari docker-compose

    ```
    docker-compose --version
    ```
    -lalu buat folder baru di sistem,dan masuk ke folder yang barusan dibuat

    -kemudian buat file docker-compose.yml

    file docker-compose.yml ini fungsinya untuk lebih mempermudah dalam memanage container,image,maupun yang lainnya,dengan docker-compose kita dapat melakukan up,down,start,stop cukup sekali tidak perlu satu-satu dulu terhadap container
    karena didalam docker-compose dapat menyimpan banyak container yang nantinya jika di up akan membuat container secara otomatis

    ```
    vi docker-compose.yml
    ```
    untuk contoh isi dari docker-compose.yml nya seperti dibawah ini;

    ```
    version: "2.2"`

    services:
      mongo:
        container_name: mongo-compose
        image: mongo:4-xenial
        ports:
          - '27017:27017'
        networks:
          - redis_network
      redis:
        container_name: redis
        image: redis:5
        ports:
          - '6379:6379'
        networks:
          - redis_network
      mongo-express:
        container_name: mongo-express
        image: mongo-express
        restart: always
        ports:
          - '27018:27017'
        depends_on:
          - redis
          - mongo
        environment:
          - NAME=Docker
          - MONGO_HOST=mongo
          - MONGO_PORT=27017
          - REDIS_HOST=redis
          - REDIS_PORT=6379
          - MONGO_INITDB_ROOT_USERNAME=compose
          - MONGO_INITDB_ROOT_PASSWORD=compose
          - ME_CONFIG_MONGODB_ADMINUSERNAME=compose
          - ME_CONFIG_MONGODB_ADMINPASSWORD=compose
          - ME_CONFIG_MONGODB_URL=mongodb://compose:compose@mongo-express:27017/
        networks:
          - redis_network

    networks:
      redis_network:
        name: redis_network
    ```

    untuk isi dari docker-compose seperti yang ada diatas,jadi bisa di edit sesuai dengan kebutuhan aja











