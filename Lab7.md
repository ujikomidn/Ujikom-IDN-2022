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

NOTE: RUNNING ALL PARTS OF THIS SCRIPT IS RECOMMENDED FOR ALL MariaDB
      SERVERS IN PRODUCTION USE!  PLEASE READ EACH STEP CAREFULLY!

In order to log into MariaDB to secure it, we'll need the current
password for the root user.  If you've just installed MariaDB, and
you haven't set the root password yet, the password will be blank,
so you should just press enter here.

Enter current password for root (enter for none): <masukkan password>
OK, successfully used password, moving on...

Setting the root password ensures that nobody can log into the MariaDB
root user without the proper authorisation.

You already have a root password set, so you can safely answer 'n'.

Change the root password? [Y/n] n
 ... skipping.

By default, a MariaDB installation has an anonymous user, allowing anyone
to log into MariaDB without having to have a user account created for
them.  This is intended only for testing, and to make the installation
go a bit smoother.  You should remove them before moving into a
production environment.

Remove anonymous users? [Y/n] y
 ... Success!

Normally, root should only be allowed to connect from 'localhost'.  This
ensures that someone cannot guess at the root password from the network.

Disallow root login remotely? [Y/n] n
 ... skipping.

By default, MariaDB comes with a database named 'test' that anyone can
access.  This is also intended only for testing, and should be removed
before moving into a production environment.

Remove test database and access to it? [Y/n] y
 - Dropping test database...
 ... Success!
 - Removing privileges on test database...
 ... Success!

Reloading the privilege tables will ensure that all changes made so far
will take effect immediately.

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
MariaDB [(none)]> create database idndb character set utf8 collate utf8_bin;
MariaDB [(none)]> grant all privileges on idndb.* to 'usermantab'@'localhost' identified by 'farrosjoss';
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
>>>DBName=idndb
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
