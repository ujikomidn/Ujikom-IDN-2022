# Lab 6

Konfigurasi untuk Lab 6 Ujikom

By: A Sunnah & Andiama
## Table of Contents
- [Step by step WLAN](#step-by-step-wlan)
- [Setting DHCP Client dan DHCP Server](#setting-dhcp-client-dan-dhcp-server)
- [Mematikan akses WinBox bagi Client 1](#mematikan-akses-winbox-bagi-client-1)
- [Setup Hotspot untuk Client](#setup-hotspot-untuk-client)
- [Blocking Detik.com](#blocking-detikcom)

## Step by step WLAN

Scan Network > Start > Connect ke AP > centang semua Security Profile + Insert Password


## Setting DHCP Client dan DHCP Server
```
ip dhcp-client add interface=wlan1 
ip address add address=172.16.10.1/24 interface=ether2
ip address add address=172.16.20.1/24 interface=ether3
ip pool add name=dhcp_pool0 ranges=172.16.10.11-172.16.10.254
ip pool add name=dhcp_pool1 ranges=172.16.20.11-172.16.20.254
ip dhcp-server add address-pool=dhcp_pool0 disabled=no interface=ether2 name=dhcp1
ip dhcp-server add address-pool=dhcp_pool1 disabled=no interface=ether3 name=dhcp2
ip dhcp-server network add address=172.16.10.0/24 dns-server=8.8.8.8 gateway=172.16.10.1
ip dhcp-server network add address=172.16.20.0/24 dns-server=8.8.8.8 gateway=172.16.20.1
```

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
ip firewall address-list add address=203.190.242.211 list=detik.com
ip firewall address-list add address=103.49.221.211 list=detik.com
ip firewall filter add chain=forward dst-address-list=detik.com action=drop
```
