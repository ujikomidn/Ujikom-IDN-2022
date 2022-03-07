# Lab 6

Konfigurasi untuk Lab 6 Ujikom

By: A Sunnah & Andiama

## Step by step WLAN

Scan Network > Start > Connect ke AP > centang semua Security Profile + Insert Password


## Setting DHCP Client & DHCP Server
```
ip dhcp-client add interface=wlan1 
ip address add address=172.16.10.1/24 interface=ether2
ip address add address=172.16.20.1/24 interface=ether3
```
DHCP Setup > ether2 > Address to Give Out: 11-255 > DNS Server 8.8.8.8 > ulangi untuk ether3


## Mematikan akses WinBox bagi Client 1
```
tool mac-server mac-winbox set allowed-interface-list=none
ip firewall filter add chain=input  src-address=!172.16.10.254 protocol=tcp dst-port=8291 action=drop
```

## Setup Hotspot untuk Client
Hotspot Setup > ether2 > DNS Name:(bebas)

## Blocking Detik.com
Buka cmd > nslookup detik.com
```
ip firewall address-list add address=203.190.242.211               
list: detik.com
ip firewall address-list add address=103.49.221.211 
list: detik.com
```