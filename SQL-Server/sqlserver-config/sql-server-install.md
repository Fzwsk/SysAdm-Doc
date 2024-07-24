# Install SQL Server 

## Environment

- RHEL 8.0 - 8.6 machine
- At least 2 GB of memory

## Step

1. melakukan update OS dengan command

   ```
   yum update -y
   ```

2. Download the SQL Server 2022 (16.x) Red Hat repository, dengan command

   ```
   sudo curl -o /etc/yum.repos.d/mssql-server.repo https://packages.microsoft.com/config/rhel/8/mssql-server-2022.repo
   ```

3. Kemudian install SQL Server dengan command

   ```
   sudo yum install -y mssql-server
   ```

4. Setelah proses instalasi selesai, jalankan `mssql-conf setup` dengan full-path

   ```
   sudo /opt/mssql/bin/mssql-conf setup
   ```

   ```
   [root@dev-sqlserver ~]# sudo /opt/mssql/bin/mssql-conf setup
   Choose an edition of SQL Server:
   1) Evaluation (free, no production use rights, 180-day limit)
   2) Developer (free, no production use rights)
   3) Express (free)
   4) Web (PAID)
   5) Standard (PAID)
   6) Enterprise (PAID) - CPU core utilization restricted to 20 physical/40 hyperthreaded
   7) Enterprise Core (PAID) - CPU core utilization up to Operating System Maximum
   8) I bought a license through a retail sales channel and have a product key to enter.
   9) Standard (Billed through Azure) - Use pay-as-you-go billing through Azure.
   10) Enterprise Core (Billed through Azure) - Use pay-as-you-go billing through Azure.

   Details about editions can be found at
   https://go.microsoft.com/fwlink/?LinkId=2109348&clcid=0x409

   Use of PAID editions of this software requires separate licensing through a
   Microsoft Volume Licensing program.
   By choosing a PAID edition, you are verifying that you have the appropriate
   number of licenses in place to install and run this software.
   By choosing an edition billed Pay-As-You-Go through Azure, you are verifying
   that the server and SQL Server will be connected to Azure by installing the
   management agent and Azure extension for SQL Server.

   Enter your edition(1-10): 2
   The license terms for this product can be found in
   /usr/share/doc/mssql-server or downloaded from:
   https://go.microsoft.com/fwlink/?LinkId=2104294&clcid=0x409

   The privacy statement can be viewed at:
   https://go.microsoft.com/fwlink/?LinkId=853010&clcid=0x409

   Do you accept the license terms? [Yes/No]:yes

   Enter the SQL Server system administrator password:
   Confirm the SQL Server system administrator password:
   Configuring SQL Server...

   ForceFlush is enabled for this instance.
   ForceFlush feature is enabled for log durability.
   Created symlink /etc/systemd/system/multi-user.target.wants/mssql-server.service → /usr/lib/systemd/system/ mssql-server.service.
   Setup has completed successfully. SQL Server is now starting.
   ```

5. Lalu cek status SQL Server sudah running atau belum dengan command

   ```
   systemctl status mssql-server
   ```

6. Untuk membuat database, perlu mendownload tool yang dapat running Transact-SQL statements di SQL Server. Berikut untuk step install 

   ```
   sudo curl -o /etc/yum.repos.d/msprod.repo https://packages.microsoft.com/config/rhel/8/prod.repo
   ```

   ```
   sudo yum install -y mssql-tools unixODBC-devel
   ```

   ```
   echo 'export PATH="$PATH:/opt/mssql-tools/bin"' >> ~/.bash_profile
   ```

   ```
   echo 'export PATH="$PATH:/opt/mssql-tools/bin"' >> ~/.bashrc
   source ~/.bashrc
   ```

7. Untuk melakukan tes masuk ke shell SQL Server localhost gunakan command

   ```
   sqlcmd -S localhost -U sa -P '<YourPassword>'
   ```

   apabila berhasil konek maka akan muncul `1>`

8. Tes melakukan create Database

   ```
   CREATE DATABASE TestDB;
   ```

   ```
   SELECT Name from sys.databases;
   ```

   Perlu di ingat di SQL Server command diatas tidak di eksekusi secara langsung perlu running command dibawah agar 2 command diatas di eksekusi

   ```
   GO
   ```

   contoh lengkap

   ```
   [root@dev-sqlserver ~]# sqlcmd -S localhost -U sa -P 'password-admin'
   1> CREATE DATABASE TestDB;
   2> SELECT Name from sys.databases;
   3> GO
   Name
   --------------------------------------------------------------------------------------------------------------------------------
   master
   tempdb
   model
   msdb
   TestDB

   (5 rows affected)
   1>
   2> QUIT
   [root@dev-sqlserver ~]#
   ```

## Reference 

- [https://learn.microsoft.com/en-us/sql/linux/quickstart-install-connect-red-hat?view=sql-server-ver16](https://learn.microsoft.com/en-us/sql/linux/quickstart-install-connect-red-hat?view=sql-server-ver16)