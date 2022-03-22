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
aidan@ususbuntu:~$ ip a
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
    inet6 ::1/128 scope host
       valid_lft forever preferred_lft forever
2: ens3: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc fq_codel state UP group default qlen 1000
    link/ether 00:50:00:00:01:00 brd ff:ff:ff:ff:ff:ff
    inet 192.168.111.140/24 brd 192.168.111.255 scope global dynamic ens3
       valid_lft 1745sec preferred_lft 1745sec
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

Kemudian pastikan service OpenVPN selalu menyala saat startup.
```
sudo systemctl enable openvpn-server@server.service
sudo systemctl start openvpn-server@server.service
```

Terakhir, ambil file .ovpn dari server tersebut ke PC kita menggunakan command seperti dibawah

```
C:\Users\Andie>ssh aidan@192.168.111.140 "sudo -S cat /root/andie.ovpn" > andie.ovpn
aidan@192.168.111.140's password:
[sudo] password for aidan: 
```

## Install DNS Server
```
sudo apt install bind9
sudo ufw allow 53
udo systemctl start bind9.service
```

Kemudian kita ganti DHCPnya menjadi Static IP
```
sudo nano /etc/netplan/00-installer-config.yaml
********Ubah jadi seperti dibawah ini********
# This is the network config written by 'subiquity'
network:
  ethernets:
    enp0s3:
      dhcp4: false
      addresses: [192.168.111.140/24]
      gateway4: 192.168.111.1
      nameservers:
        search: [andieidn.com]
        addresses: [192.168.111.254, 192.168.111.1]
  version: 2
*********************************************


sudo nano /etc/resolv.conf
********Ubah jadi seperti dibawah ini********
nameserver 192.168.111.254
nameserver 192.168.111.1
options edns0
search andieidn.com
*********************************************


sudo nano /etc/hosts
********Ubah jadi seperti dibawah ini********
127.0.0.1 localhost
127.0.1.1 ususbuntu
192.168.111.140 andieidn.com
*********************************************
```

Kemudian konfigurasikan DNS Server tersebut
```
sudo nano /etc/bind/named.conf.local
*****************Buat seperti dibawah tersebut*****************
// Consider adding the 1918 zones here, if they are not used in your
// organization
//include "/etc/bind/zones.rfc1918";

zone "andieidn.com" {
        type master;
        file "/etc/bind/db.andieidn";
};
***************************************************************


sudo cp /etc/bind/db.local /etc/bind/db.andieidn
sudo nano /etc/bind/db.andieidn
*****************Buat seperti dibawah tersebut*****************
;
; BIND data file for Andie IDN
;
$TTL    604800
@       IN      SOA     ns.andieidn.com. root.andieidn.com. (
                              2         ; Serial
                         604800         ; Refresh
                          86400         ; Retry
                        2419200         ; Expire
                         604800 )       ; Negative Cache TTL
;
@       IN      NS      ns.andieidn.com.
@       IN      A       192.168.111.140
@       IN      MX      10      mail.andieidn.com.
ns      IN      A       192.168.111.140
www     IN      CNAME   ns
mail    IN      A       192.168.111.140
***************************************************************
```

Lalu kita tambahkan Reverse Zone
```
sudo nano /etc/bind/named.conf.local
*****************Buat seperti dibawah tersebut*****************
// Consider adding the 1918 zones here, if they are not used in your
// organization
//include "/etc/bind/zones.rfc1918";

zone "andieidn.com" {
        type master;
        file "/etc/bind/db.andieidn";
};

zone "22.168.192.in-addr.arpa" {
        type master;
        file "/etc/bind/db.192";
};
***************************************************************


sudo cp /etc/bind/db.127 /etc/bind/db.192
sudo nano /etc/bind/db.192
*****************Buat seperti dibawah tersebut*****************
;
; BIND reverse data file for Andie IDN
;
$TTL    604800
@       IN      SOA     ns.andieidn.com. root.andieidn.com. (
                              2         ; Serial
                         604800         ; Refresh
                          86400         ; Retry
                        2419200         ; Expire
                         604800 )       ; Negative Cache TTL
;
@       IN      NS      ns.andieidn.com.
1       IN      PTR     ns.andieidn.com.
1       IN      PTR     www.andieidn.com
1       IN      PTR     mail.andieidn.com
***************************************************************
```


Selanjutnya setting DNS Caching
```
sudo nano /etc/bind/named.conf.options
********************Edit seperti di bawah ini********************
        // If your ISP provided one or more IP addresses for stable
        // nameservers, you probably want to use them as forwarders.
        // Uncomment the following block, and insert the addresses replacing
        // the all-0's placeholder.

         forwarders {
              9.9.9.9;
              1.1.1.1;
        };
*****************************************************************
```

Terakhir, restart service BIND9
```
systemctl restart bind9.service
```
## Install File Server
```
sudo apt install samba
sudo systemctl enable smbd
sudo systemctl start smbd
```

Backup konfigurasi default yang dimiliki oleh samba dan buat folder yang akan dipakai untuk berbagi file
```
cp /etc/samba/smb.conf /etc/samba/smb.conf.ori
sudo mkdir -p /srv/samba/share
sudo chown nobody:nogroup /srv/samba/share/
```

Lalu edit file smb.conf
```
sudo nano /etc/samba/smb.conf

******Taruh di paling bawah******
[Sharing_Folder]
    comment = Folder untuk berbagi konten
    path = /srv/samba/share
    browsable = yes
    guest ok = yes
    read only = no
    create mask = 0755
```

Coba akses ke Server menggunakan format "\\(IP Address)" pada File Explorer

![image](https://user-images.githubusercontent.com/100014814/159407448-cd51f38a-bb60-4b3a-90d1-fe460450d608.png)

Jika sudah ada Folder "Sharing_Folder", maka konfigurasi Samba kalian sudah berhasil.

![image](https://user-images.githubusercontent.com/100014814/159407559-a37bbff0-031f-4e44-ac63-d66e0cbb56df.png)
