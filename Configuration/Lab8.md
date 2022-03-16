# Lab 8
Konfigurasi untuk Lab 8 Ujikom

By: Andiama

## Install SSH Server
Instalasi SSH Server sangat mudah, seperti dibawah
```
sudo apt update
sudo apt install openssh-server
sudo systemctl enable ssh
sudo systemctl start ssh
```

## Install FTP Server
```
sudo apt install vsftpd
sudo systemctl enable vsftpd
sudo systemctl start vsftpd
sudo nano /etc/vsftpd.conf
>>>write_enable=YES
```

## Install Web Server
```
sudo apt install apache2
sudo systemctl enable apache2
sudo systemctl start apache2
```

Jika sudah, cek menggunakan command "hostname -I".

Setelah itu, coba masukkan IP yang tertera pada Device yang terhubung.
Jika sudah, akan terlihat seperti dibawah ini
![image](https://user-images.githubusercontent.com/100014814/158320238-bf777677-a4a0-4266-aad2-8692b1575485.png)

## Install VPN Server
Pertama tama, cari tahu IP address Server Ubuntu kalian
```
ip a
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
    inet6 ::1/128 scope host
       valid_lft forever preferred_lft forever
2: ens3: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc fq_codel state UP group default qlen 1000
    link/ether 00:50:00:00:01:00 brd ff:ff:ff:ff:ff:ff
    inet ******192.168.111.140/24****** brd 192.168.111.255 scope global dynamic ens3
       valid_lft 997sec preferred_lft 997sec
    inet6 fe80::250:ff:fe00:100/64 scope link
       valid_lft forever preferred_lft forever
```

Setelah mendapat IP Addressnya, install service OpenVPN menggunakan wget, lalu install service tersebut
```
wget https://git.io/vpn -O openvpn-ubuntu-install.sh
chmod -v +x openvpn-ubuntu-install.sh
sudo ./openvpn-ubuntu-install.sh

Welcome to this OpenVPN road warrior installer!

This server is behind NAT. What is the public IPv4 address or hostname?
Public IPv4 address / hostname [103.144.175.62]:

Which protocol should OpenVPN use?
   1) UDP (recommended)
   2) TCP
Protocol [1]:

What port should OpenVPN listen to?
Port [1194]:

Select a DNS server for the clients:
   1) Current system resolvers
   2) Google
   3) 1.1.1.1
   4) OpenDNS
   5) Quad9
   6) AdGuard
DNS server [1]: 3

Enter a name for the first client:
Name [client]: andie

OpenVPN installation is ready to begin.
Press any key to continue...
```

Terakhir, pastikan service OpenVPN selalu menyala saat startup
```
sudo systemctl enable openvpn-server@server.service
sudo systemctl start openvpn-server@server.service
```

## Install File Server
```
sudo apt install samba
sudo systemctl enable smbd
sudo systemctl start smbd
```
