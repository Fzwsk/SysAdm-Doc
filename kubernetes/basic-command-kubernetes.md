basic command kubernetes

1. melihat semua kubernetes node

   ```
   kubectl get node
   ```
2. melihat detail node

   ```
   kubectl describe node <namanode>
   ```
3. melihat semua pod dalam node 

   ```
   kubectl get pod
   ```
4. melihat detail pod

   ```
   kubectl describe pod <namapod>
   ```
5. membuat pod, namespace, probe, replica set, deamon set, job, cron job, service, ingress, multi container, configmap, secret, deployment, dll di kubernetes dengan image yg sudah di pull dari docker registry (cara ini sama dengan cara membuat resource lain di kubernetes)

   -pastikan sudah mendownload image

   -pod dapat di created ulang dengan image yg sama, asalkan nama dari pod harus beda, berikut command untuk create nya
   
   ```
   kubectl create -f namafile.yaml
   ```
6. test mengakses pod (nginx) di web browser dengan memforward port

    ```
    kubectl port-forward namapod portAkses:portPod
    kubectl port-forward namapod 8888:8080

    ```
7. label pada pod
   a. melihat label pada pod
    ```
    dkubectl get pods --show-labels
    ```
   b. menambah label pada pod
    ```
    kubectl label pod namapod key=value
    ```
   c. mengubah label pada pod
    ```
    kubectl label pod namapod key=value --overwrite
    ```
   d. mencari pod dengan label
    ```
    kubectl get pods -l key
    kubectl get pods -l key=value
    kubectl get pods -l ‘!key’
    kubectl get pods -l key!=value
    kubectl get pods -l ‘key in (value1,value2)’
    kubectl get pods -l ‘key notin (value1,value2)’ 
    ```
8. melihat namespace
   ```
   kubectl get namespaces / namespace / ns
   ```
   a. melihat pod yang ada di namespace
    ```
    kubectl get pod --namespace <namanamespace>
    ```
   b. membuat pod didalam namespace
    ```
    kubectl create -f namafile.yaml --namespace <namanamespace>
    ```
9. menghapus pod, namespace, probe, replica set, deamon set, job, cron job, service, ingress, multi container, configmap, secret, deployment, dll di kubernetes

   -untuk menghapus pod yg ada didalam namespace, service, atau replica set pastikan menginput resource nya dulu sebelum mengeksekusi perintah delete

   ```
   kubectl delete pod <namapod>
   kubectl delete pod <namapod> --namespace <namespace>

   kubectl delete service <namaservice> , namespace <namespace>, replicaset <namars>, deamonset <namads>, ingress <namaingress>, dll
   ```
10. melihat log aplikasi di container kita

   -fungsinya untuk mempermudah pada saat debuging,agar lebih mudah untuk mengetahui letak erornya dimana docker container logs namacontainer

   -jika ingin mengetahui logs yg terbaru dari sebuah container maka tambahkan flag -f

    ```
    kubectl logs <namapod>
    ```
11. menggunakan command exec -it

    -command ini untuk mengeksekusi kode program yg ada di dalam pod/container

    -bukan hanya mengeksekusi saja, tapi bisa masuk ke dalam kubernetes pod/container melalui image

     ```
     docker exec -it namapod /bin/sh
     ```
     - /bin/sh itu artinya akan masuk ke folder /bin/sh dan akan mengeksekusi program kode image yang ada didalam pod/container

     -i artinya argument interaktif,agar inputnya tetap aktif

     -t artinya argument agar dapat alokasi ke terminal akses