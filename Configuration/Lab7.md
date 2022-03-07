# Lab 7

Konfigurasi untuk Lab 7 Ujikom

Made by: Andiama

## Zabbix Installation

### LAMP Server Installation
LAMP merupakan singkatan dari Linux, Apache, MariaDB dan PHP. Mari kita install semuanya dengan 2 command.
```
sudo apt update && sudo apt upgrade
sudo apt install apache2 php php-mysql php-mysqlnd php-ldap php-bcmath php-mbstring php-gd php-pdo php-xml libapache2-mod-php
```

Selanjutnya, ubah beberapa hal dalam "php.ini". Kalian bisa menggunakan F6 ataupun CTRL + W untuk mencari line yang diperlukan
```
sudo nano /etc/php/7.4/apache2/php.ini
>>>post_max_size = 16M
>>>upload_max_filesize = 2M
>>>max_execution_time = 300
>>>max_input_time = 300
>>>memory_limit = 128M
>>>session.auto_start = 0
>>>mbstring.func_overload = 0
>>>date.timezone = Asia/Jakarta
```

Kemudian restart daemon dari apache2 agar apache2 mendetect perubahan pada PHP
```
sudo systemctl restart apache2.service	
```

### MariaDB Installation
Install Library dan Database untuk MariaDB agar Zabbix dapat bekerja.
```
sudo apt-get install mariadb-server mariadb-client libmariadb-dev
```

Setelah selesai instalasi package MariaDB, run daemon untuk MariaDB lalu install menggunakan "mysql_secure_installation"

Ikutin aja kayak dibawah, insyaallah work 100% no hack no root indo sub kalo ikutin step by stepnya

*(BTW inget password root ya, jangan sampe lupa, simpen di notepad kalo misalnya kalian menderita demensia ato apalah itu)*
```
aidan@ubuntu:~$ sudo mysql_secure_installation

Enter current password for root (enter for none): <masukkan password>

You already have a root password set, so you can safely answer 'n'.

Change the root password? [Y/n] n
 ... skipping.

Remove anonymous users? [Y/n] y
 ... Success!

Disallow root login remotely? [Y/n] n
 ... skipping.

Remove test database and access to it? [Y/n] y
 - Dropping test database...
 ... Success!
 - Removing privileges on test database...
 ... Success!

Reload privilege tables now? [Y/n] y
 ... Success!

Cleaning up...

All done!  If you've completed all of the above steps, your MariaDB
installation should now be secure.

Thanks for using MariaDB!
```
Next, buat database, user dan password untuk MariaDB kita.
```
sudo mysql -u root -p
MariaDB [(none)]> create database idnzabbix character set utf8 collate utf8_bin;
MariaDB [(none)]> grant all privileges on idnzabbix.* to 'usermantab'@'localhost' identified by 'farrosjoss';
MariaDB [(none)]> flush privileges;
MariaDB [(none)]> exit
```

### Zabbix Server Installation
Ambil dulu package Zabbix dari repo mereka, kemudian install Zabbix
```
sudo wget https://repo.zabbix.com/zabbix/5.4/ubuntu/pool/main/z/zabbix-release/zabbix-release_5.4-1+ubuntu20.04_all.deb
sudo dpkg -i zabbix-release_5.4-1+ubuntu20.04_all.deb
sudo apt update
sudo apt install zabbix-server-mysql zabbix-frontend-php zabbix-apache-conf zabbix-sql-scripts zabbix-agent
```
Lalu restart daemon untuk apache2 agar perubahan dari Zabbix dapat terdeteksi
```
sudo systemctl restart apache2
```


#### Zabbix Server Configuration
Import database dari Zabbix kita agar digunakan oleh MariaDB
```
sudo zcat /usr/share/doc/zabbix-sql-scripts/mysql/create.sql.gz | mysql -u usermantab idndb -p
Enter password: farrosjoss
```
Selanjutnya, ubah konfigurasi untuk Server Zabbix
```
sudo nano /etc/zabbix/zabbix_server.conf
>>>DBHost=localhost
>>>DBName=idnzabbix
>>>DBUser=usermantab
>>>DBPassword=farrosjoss
```
Terakhir, restart service untuk Zabbix Server
```
sudo systemctl restart zabbix-server.service
```


#### Zabbix Agent Configuration
Untuk Agent-nya, kita hanya perlu mengubah konfigurasinya saja.
```
sudo nano /etc/zabbix/zabbix_agentd.conf 
>>>Server=127.0.0.1
>>>ListenPort=10050
```
Lalu restart service Zabbix Agent
```
sudo systemctl restart zabbix-agent.service 
```

### Zabbix Frontend Installation
Sekarang, buka IP VM kalian lalu "/zabbix". Semisal IP VM saya adalah 192.168.111.143, maka buka di browser "192.168.111.143/zabbix"

Jika sudah, maka akan terlihat seperti ini:
![image](https://user-images.githubusercontent.com/100014814/156908374-2d4b6bdb-01b1-4b8a-9d75-e75136e41677.png)

Bila kalian mengikuti semua step sebelumnya, harusnya pada bagian ini sudah tercentang semuanya
![image](https://user-images.githubusercontent.com/100014814/156908398-e1fa62ac-08b4-45a0-a3fc-26660b11d473.png)

Isi bagian ini dengan kredensial yang telah kita tambahkan sebelumnya
![image](https://user-images.githubusercontent.com/100014814/156908443-ac9c2524-8fba-4281-91f0-cfc176c4e1b6.png)

Kalian bebas mengisi bagian yang ini dengan nama apapun
![image](https://user-images.githubusercontent.com/100014814/156908491-701b0d13-a5c7-4a2a-8928-61ca3dc850bf.png)

Pada bagian ini, konfigurasikan Zabbix agar sesuai dengan Zona Waktu yang dimiliki beserta tema yang ingin digunakan
![image](https://user-images.githubusercontent.com/100014814/156908561-a561fb55-b6a3-4576-b6cd-fc77a835ad7d.png)

Pastikan semuanya sudah sesuai. Jika benar, klik "next step"
![image](https://user-images.githubusercontent.com/100014814/156908572-9e0f10fb-2c86-4b32-8d81-8777fd346afa.png)

Login ke dalam Zabbix menggunakan kredensial default, yaitu "Admin" sebagai user, dan "zabbix" sebagai password
![image](https://user-images.githubusercontent.com/100014814/156908634-e6c0014f-6a6f-416f-82c3-7fb6db60a935.png)

Jika sudah masuk, akan seperti ini interface Zabbixnya
![image](https://user-images.githubusercontent.com/100014814/156908661-643c52fb-bba2-4780-badb-85838a280013.png)

Apabila kalian ingin mengubah password, bisa langsung ke
**Administration > Users > Admin**
![image](https://user-images.githubusercontent.com/100014814/156908768-87ee4f7d-c392-443e-823a-2c073c21597b.png)

Disini, kalian dapat mengubah username, password, maupun nama pemilik akun tersebut
![image](https://user-images.githubusercontent.com/100014814/156908807-c5906b13-1b38-40a2-960e-3b1659cb4e76.png)


## Cacti Installation
Untuk Cacti, kita tinggal perlu mengedit beberapa hal.
```
sudo nano /etc/mysql/mariadb.conf.d/50-server.cnf
>>>collation-server = utf8mb4_unicode_ci
>>>tmp_table_size = 64M
>>>join_buffer_size = 64M
>>>innodb_file_format = Barracuda
>>>innodb_large_prefix = 1
>>>max_heap_table_size = 128M
>>>innodb_io_capacity = 5000
>>>innodb_io_capacity_max = 10000
>>>innodb_buffer_pool_size = 512M
>>>innodb_flush_log_at_timeout = 3
>>>innodb_read_io_threads = 32
>>>innodb_write_io_threads = 16
sudo systemctl restart mariadb
```
Lalu buat Database dan masukkan user yang telah kita buat sebelumnya pada MySQL
```
sudo mysql
MariaDB [(none)]> create database idncacti character set utf8mb4 collate utf8mb4_unicode_ci;
MariaDB [(none)]> GRANT ALL ON idncacti.* TO usermantab@localhost identified by 'farrosjoss';
MariaDB [(none)]> flush privileges;
MariaDB [(none)]> exit
```
Lalu masukkan Data untuk Time Zone ke dalam MySQL
```
sudo mysql mysql < /usr/share/mysql/mysql_test_data_timezone.sql  
sudo mysql
MariaDB [(none)]> grant select on mysql.time_zone_name to usermantab@localhost;
MariaDB [(none)]> flush privileges;
MariaDB [(none)]> exit
```
Selanjutnya, install Cacti beserta dependencynya
```
sudo apt install rrdtool snmp snmpd snmp-mibs-downloader libsnmp-dev
sudo wget https://www.cacti.net/downloads/cacti-latest.tar.gz
sudo mkdir /var/www/html/cacti
sudo tar xzf cacti-latest.tar.gz -C /var/www/html/cacti
sudo mv /var/www/html/cacti/cacti-1.2.19/* /var/www/html/cacti/
sudo mysql cactidb < /var/www/html/cacti/cacti.sql
sudo chown -R www-data:www-data /var/www/html/cacti/
sudo chmod -R 775 /var/www/html/cacti/
```
Setelah itu, konfigurasikan agar Cacti menggunakan database yang telah kita berikan
```
sudo nano /var/www/html/cacti/include/config.php

********Edit data agar seperti dibawah ini********
$database_type = 'mysql';
$database_default = 'idncacti';
$database_hostname = 'localhost';
$database_username = 'usermantab';
$database_password = 'farrosjoss';
$database_port = '3306';
```
Kemudian konfigurasikan Apache agar dapat berjalan untuk Cacti
```
sudo nano /etc/apache2/sites-available/cacti.conf

********Tambahkan line dibawah ke dalam cacti.conf yang baru dibuat********
Alias /cacti    /var/www/html/cacti
<Directory /var/www/html/cacti/>
<IfModule mod_authz_core.c>
Require all granted
</IfModule>
</Directory>
```
Terakhir, test Apache untuk Cacti. Jika outputnya "Syntax OK", maka langsung restart daemon Apache
```
sudo apachectl configtest
sudo systemctl restart apache2
```
**Jika mendapat error "Could not reliably determine the server's fully qualified domain name, using 127.0.1.1. Set the 'ServerName' directive globally to suppress this message", lakukan seperti dibawah ini baru lakukan hal diatas*
```
sudo nano /etc/apache2/apache2.conf

********Tambahkan line dibawah ke bagian paling bawah konfigurasi Apache********
ServerName localhost
```
## SNMP on Mikrotik
Pertama tama, tambahkan IP Address agar dapat di ping oleh kedua PC
```
ip address add address=192.168.10.1/24 interface=ether1
ip address add address=192.168.20.1/24 interface=ether2
```
Kemudian tambahkan setting SNMP pada MikroTik
```
[admin@MikroTik] > snmp set enabled=yes
[admin@MikroTik] > snmp community set name=SNMPIDN 0
[admin@MikroTik] > snmp community print             
Flags: * - default, X - disabled 
 #    NAME     ADDRESSES                                      SECURITY   READ-ACCESS WRITE-ACCESS
 0 *  SNMPIDN  ::/0                                           none       yes         no  
[admin@MikroTik] > snmp print
          enabled: yes
          contact: 
         location: 
        engine-id: 
      trap-target: 
   trap-community: SNMPIDN
     trap-version: 1
  trap-generators: temp-exception
```
Lalu pada Server Linux, install dan coba SNMP
```
aidan@ubuntu:~$ sudo apt-get install snmp
aidan@ubuntu:~$ snmpwalk -v2c -c SNMPIDN 192.168.10.1
```
Pada server Zabbix, pergi ke
**Configuration > Hosts > Create Host**
![image](https://user-images.githubusercontent.com/100014814/156949568-c2ad96dc-b367-42a5-a585-75907bfc7a7b.png)

Kemudian buat konfigurasi seperti dibawah
![image](https://user-images.githubusercontent.com/100014814/156949706-60f50df6-95d0-46b4-b9a0-83907113de3d.png)

Lalu tambahkan macro untuk SNMP MikroTik pada menu
**Macro**
![image](https://user-images.githubusercontent.com/100014814/156950300-b036cbdc-7e8e-48b2-98c9-9f83cbb3bda3.png)

Terakhir, tambahkan Template MikroTik SNMP pada menu
**Templates**
![image](https://user-images.githubusercontent.com/100014814/156950892-819da6ad-38fc-4dbe-97ec-82836851c059.png)
