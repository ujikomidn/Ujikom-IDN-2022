# Lab 8
Konfigurasi untuk Lab 8 Ujikom

By: Andiama

## Table of Contents
- [Server Configuration](#server-configuration)
  * [Install SSH Server](#install-ssh-server)
  * [Install FTP Server](#install-ftp-server)
  * [Install DNS Server](#install-dns-server)
  * [Install DHCP Server](#install-dhcp-server)

## Server Configuration

### Install SSH Server
Instalasi SSH Server sangat mudah, seperti dibawah
```
sudo apt update
sudo apt install openssh-server
sudo systemctl enable ssh
sudo systemctl start ssh
```

### Install FTP Server
```
sudo apt install vsftpd
sudo systemctl enable vsftpd
sudo systemctl start vsftpd
sudo nano /etc/vsftpd.conf
>>>write_enable=YES
```

### Install DNS Server
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
      addresses: [192.168.111.10/24]
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
192.168.111.10 andieidn.com
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
@       IN      A       192.168.111.10
@       IN      MX      10      mail.andieidn.com.
ns      IN      A       192.168.111.10
www     IN      CNAME   ns
mail    IN      A       192.168.111.10
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

### Install DHCP Server
```
sudo apt install isc-dhcp-server
```

Edit file "dhcpd.conf" menjadi seperti dibawah
```
sudo nano /etc/dhcp/dhcpd.conf

*********Tambahkan tanda pagar pada opsi dibawah ini*********
# option definitions common to all supported networks...
# option domain-name "example.org";
# option domain-name-servers ns1.example.org, ns2.example.org;

# default-lease-time 600;
# max-lease-time 7200;

****Hilangkan tanda pagar pada opsi dan ubah menjadi seperti dibawah****
# A slightly different configuration for an internal subnet.
subnet 192.168.111.0 netmask 255.255.255.0 {
  range 192.168.111.11 192.168.111.111;
  option domain-name-servers 192.168.111.254, 192.168.111.1;
  option domain-name "andieidn.com";
  option subnet-mask 255.255.255.0;
  option routers 192.168.111.1;
  option broadcast-address 192.168.111.255;
  default-lease-time 600;
  max-lease-time 7200;
}
************************************************************************
```

Dan edit file untuk DHCP Server
```
sudo nano /etc/default/isc-dhcp-server

*******Tambahkan Interface yang akan dipakai untuk memberikan IP DHCP*******
# On what interfaces should the DHCP server (dhcpd) serve DHCP requests?
#       Separate multiple interfaces with spaces, e.g. "eth0 eth1".
INTERFACESv4="ens3"
INTERFACESv6=""
****************************************************************************
```

Terakhir, restart Service DHCP
```
sudo systemctl restart isc-dhcp-server.service
```
