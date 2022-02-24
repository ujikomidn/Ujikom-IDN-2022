# Lab 7

Konfigurasi untuk Lab 7 Ujikom

## Step By Step Zabbix Installation

### Install Apache
Upgrade system kemudian reboot VM
```
sudo apt update && sudo apt -y full-upgrade
sudo reboot
```
Kemudian install Apache dan ubah Line ke-26 menjadi "ServerTokens Prod"
```
sudo apt update
sudo apt install apache2
sudo nano /etc/apache2/conf-enabled/security.conf

>>>ServerTokens Prod
```
Set ServerName dan ServerAdmin, kemudian restart service dari Apache
```
sudo nano /etc/apache2/apache2.conf

>>>ServerName zabbix.ujikom.com
>>>ServerAdmin admin@ujikom.com

sudo systemctl restart apache2
```
**Opsional* Buat Rule untuk HTTP dan HTTPS jika menggunakan Firewall UFW
```
sudo ufw allow http
sudo ufw allow https
```

### Install PHP
```
sudo apt -y install php php-cgi php-common libapache2-mod-php php-mbstring php-net-socket php-gd php-xml-util php-mysql php-bcmath
```
Cek versi PHP menggunakan command "php -v"
```
aidan@overlord:~$ php -v
PHP 7.4.3 (cli) (built: Nov 25 2021 23:16:22) ( NTS )
Copyright (c) The PHP Group
Zend Engine v3.4.0, Copyright (c) Zend Technologies
    with Zend OPcache v7.4.3, Copyright (c), by Zend Technologies
```
Konfigurasikan agar Apache terhubung dengan PHP, kemudian ubah Time Zone menjadi Time Zone lokal, dan terakhir restart service Apache (Konfigurasi Time Zone letaknya di Line ke-962, jadi pesan dari gua; Enjoy your trip to Line 962!)
```
sudo a2enconf php7.*-cgi
sudo grep -n date.timezone /etc/php/*/apache2/php.ini

>>>961:; http://php.net/date.timezone
>>>962:;date.timezone =

sudo nano /etc/php/*/apache2/php.ini

>>>[Date]
>>>; Defines the default timezone used by the date functions
>>>; http://php.net/date.timezone
>>>;date.timezone = "Asia/Jakarta"

sudo systemctl restart apache2
```

### Install MariaDB
```
sudo apt update
sudo apt install mariadb-server
```
Setelah MariaDB sudah didapat, buka terminal MySQL lalu buatlah database dan user untuk Zabbix nantinya
```
sudo mysql -u root
>>>CREATE DATABASE zabbix character set utf8 collate utf8_bin;;
>>>GRANT ALL PRIVILEGES ON zabbix.* TO zabbix@'localhost' IDENTIFIED BY 'MQmantap';
>>>FLUSH PRIVILEGES; 
>>>QUIT 
```
