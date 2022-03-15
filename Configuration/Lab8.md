# Lab 8
Konfigurasi untuk Lab 8 Ujikom

By: Andiama

## Install SSH Server
Instalasi SSH Server sangat mudah, seperti dibawah
```
sudo apt-get install openssh-server
sudo systemctl enable ssh
sudo systemctl start ssh
```

## Install FTP Server
```
sudo apt-get install vsftpd
sudo systemctl enable vsftpd
sudo systemctl start vsftpd
sudo nano /etc/vsftpd.conf
>>>write_enable=YES
```
