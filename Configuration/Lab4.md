# Lab 4

Konfigurasi untuk Lab 4 Ujikom 

By: A Sunnah

## Table of Contents
- [Script R-1](#script-r-1)
- [Script SW-1](#script-sw-1)
- [Script SW-2](#script-sw-2)

## Topology
![image](https://user-images.githubusercontent.com/100014814/159839368-dc1c4868-82cc-4466-a01f-bd6128179125.png)

## Script R-1
```
hostname A-SUNNAH-R1

interface ethernet 0/0
no shutdown
description TO-WAN
ip address dhcp
ip nat outside

interface ethernet 0/1
no shutdown
description TO-LAN
ip nat inside

interface ethernet 0/1.10
encapsulation dot1Q 10
ip address 192.168.10.1 255.255.255.0

interface ethernet 0/1.20
encapsulation dot1Q 20
ip address 192.168.20.1 255.255.255.0

service dhcp
ip dhcp pool VLAN-10
network 192.168.10.0 255.255.255.0
default-router 192.168.10.1

ip dhcp pool VLAN-20
network 192.168.20.0 255.255.255.0
default-router 192.168.20.1

access-list 1 permit any
ip access-list extended A-SUNNAH-RULES
1 deny ip 192.168.20.0 0.0.0.255 host 119.81.42.42
100 permit ip any any

interface ethernet 0/0
ip access-group A-SUNNAH-RULES out

ip nat inside source list 1 interface Ethernet 0/0 overload
```

## Script SW-1
```
hostname A-SUNNAH-SW01

vtp mode client
vtp domain A-SUNNAH
vtp password MUPENG

interface range ethernet 0/1-2
switchport mode access
description ETHERCHANNEL-TO-SW2
channel-group 1 mode desirable

interface port-channel 1
description ETHERCHANNEL-TO-SW2
switchport trunk encapsulation dot1q
switchport mode trunk

interface ethernet 0/0
switchport trunk encapsulation dot1q
switchport mode trunk
description TO-R1
```

## Script SW-2
```
hostname A-SUNNAH-SW02

vlan 10
vlan 20

vtp mode server
vtp domain A-SUNNAH
vtp password MUPENG

interface range ethernet 0/1-2
switchport mode access
description ETHERCHANNEL-TO-SW1
channel-group 1 mode auto

interface port-channel 1
description ETHERCHANNEL-TO-SW1
switchport trunk encapsulation dot1q
switchport mode trunk

interface ethernet 0/0
switchport mode access
description TO-PC-A
switchport access vlan 10
switchport port-security
switchport port-security violation shutdown
switchport port-security mac-address sticky

interface ethernet 0/3
switchport mode access
description TO-PC-B
switchport access vlan 20
switchport port-security
switchport port-security violation restrict
switchport port-security mac-address sticky
```

[NEXT](https://github.com/ujikomidn/Ujikom-IDN-2022/blob/main/Configuration/Lab5.md)
