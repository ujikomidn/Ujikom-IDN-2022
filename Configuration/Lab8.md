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

## Install Web Server
```
sudo apt-get install apache2
```

Jika sudah, cek menggunakan command "hostname -I".

Setelah itu, coba masukkan IP yang tertera pada Device yang terhubung.
Jika sudah, akan terlihat seperti dibawah ini
![image](https://user-images.githubusercontent.com/100014814/158320238-bf777677-a4a0-4266-aad2-8692b1575485.png)
