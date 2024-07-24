# Create and Manage Administrator from FreeIPA 

# Environment

- Centos 7

- FreeIPA Server

1. create user di freeipa, pastikan sebelum create user, sudah kinit admin

   ```
   kinit admin
   ```
   kemudian baru add-user di freeipa nya

   ```
   ipa user-add namauser --first=(isikan sesuai namauser) --last=(isikan sesuai namauser) --password
   ```
2. membuat group di freeipa
   
   ```
   ipa group-add group_name
   ```
3. memasukkan user ke group di freeipa
  
   ```
   ipa group-add group_name --users=namausers
   ```
4. membuat role-add di freeipa

   ```
   ipa role-add nama_role
   ```
5. menambahkan privileges kedalam role-add di freeipa
   
   value dari privileges bisa User Administrators atau Service Administrators

   ```
   ipa role-add nama_role --privileges="User Administrators"
   ```
6. memasukkan user ke dalam role-add di freeipa

   ```
   ipa role-add nama_role --users=nama_user
   ```
7. jika sudah maka coba untuk kinit ke user nya
   
   ```
   kinit nama_user
   ```
8. jika berhasil kinit ke user, maka kinit ke admin lagi,dan tinggal get keytab nya, berikut command nya

   ```
   ipa-getkeytab -s (isikan fqdn dari server freeipa) -p (nama_user@REALMEdariFreeIPA) -k isikannamakeytabs
   ```
9. Selanjutnya bila ingin mengcopy keytab dari user tsb ke node lain, tinggal running command scp nya

   ```
   scp nama_keytab (isikanhost@ip dari node yg dituju):pathtujuan
   ```
10. Kemudian cek di node yg dituju tadi,dan bila mau mengubah kepemilikan user dari keytab nya, berikut command nya
    
    ```
    chown -R nama_user:nama_user path/nama-keytabnya
    ```

