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
sudo vim /etc/apache2/conf-enabled/security.conf
>>>ServerTokens Prod
```
Set ServerName dan ServerAdmin, kemudian restart service dari Apache
```
sudo vim /etc/apache2/apache2.conf
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
