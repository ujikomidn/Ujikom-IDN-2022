# Lab 1

Konfigurasi untuk Lab 1 Ujikom 

By: A Sunnah

## Network Scenario

| VLAN Name | IP Address |
| --------- | ---------- |
| VLAN-100: | 192.168.1.0/24 |
| VLAN-101: | 192.168.2.0/24 |
| VLAN-102: | 192.168.3.0/24 |
| VLAN-103: | 192.168.4.0/24 |

| Device Name               | IP Address |
| ------------------------- | ---------- |
| R-Core-01-Network - R-ABR:| 172.16.1.0/30 |
| R-ABR - R-ASBR-01:        | 172.16.1.4/30 |
| R-ABR - R-ASBR-02:        | 172.16.1.8/30 |
| R-ASBR-01 - R-ST-01:      | 172.16.1.12/30 |
| R-ASBR-02 - R-ST-01:      | 172.16.1.16/30 |

## Script R-Core-01-Network
```
/system identity
set name=R-Core-01-Network

/interface ethernet
set [ find default-name=ether2 ] name=TO-R-ABR
set [ find default-name=ether1 ] name=TO-WAN\

/ip address
add address=172.16.1.1/30 interface=TO-R-ABR network=172.16.1.0
/ip dhcp-client
add disabled=no interface=TO-WAN

/ip route
add distance=1 gateway=TO-WAN
/routing ospf instance
set [ find default=yes ] redistribute-static=as-type-1
/routing ospf network
add area=backbone network=172.16.1.0/30
```

## Script R-ABR
```
/system identity
set name=R-ABR

/interface ethernet
set [ find default-name=ether2 ] name=TO-R-ABSR-01
set [ find default-name=ether3 ] name=TO-R-ABSR-02
set [ find default-name=ether1 ] name=TO-R-CORE-01-NETWORK

/ip address
add address=172.16.1.2/30 interface=TO-R-CORE-01-NETWORK network=172.16.1.0
add address=172.16.1.5/30 interface=TO-R-ABSR-01 network=172.16.1.4
add address=172.16.1.9/30 interface=TO-R-ABSR-02 network=172.16.1.8

/routing ospf area
add area-id=0.0.0.1 name=AREA-01
add area-id=0.0.0.2 name=AREA-02
/routing ospf network
add area=backbone network=172.16.1.0/30
add area=AREA-01 network=172.16.1.4/30
add area=AREA-02 network=172.16.1.8/30
```

## Script R-ASBR-01
```
/system identity
set name=R-ASBR-01

/interface ethernet
set [ find default-name=ether1 ] name=TO-R-ABR
set [ find default-name=ether2 ] name=TO-R-ST-01

/ip address
add address=172.16.1.6/30 interface=TO-R-ABR network=172.16.1.4
add address=172.16.1.13/30 interface=TO-R-ST-01 network=172.16.1.12

/routing ospf instance
set [ find default=yes ] redistribute-static=as-type-1
/routing ospf area
add area-id=0.0.0.1 name=AREA-01
/routing ospf network
add area=AREA-01 network=172.16.1.4/30

/ip firewall address-list
add address=172.16.1.0/24 list=IP-INTERNAL
add address=192.168.1.0/24 list=IP-CLIENT
add address=192.168.2.0/24 list=IP-CLIENT
add address=192.168.3.0/24 list=IP-CLIENT
add address=192.168.4.0/24 list=IP-CLIENT
/ip firewall filter
add action=drop chain=input dst-address-list=IP-INTERNAL protocol=icmp src-address-list=IP-CLIENT
```

## Script R-ASBR-02
```
/system identity
set name=R-ASBR-02

/interface ethernet
set [ find default-name=ether1 ] name=TO-R-ABR
set [ find default-name=ether2 ] name=TO-R-ST-01

/ip address
add address=172.16.1.10/30 interface=TO-R-ABR network=172.16.1.8
add address=172.16.1.17/30 interface=TO-R-ST-01 network=172.16.1.16

/routing ospf instance
set [ find default=yes ] redistribute-static=as-type-1
/routing ospf area
add area-id=0.0.0.2 name=AREA-02
/routing ospf network
add area=AREA-02 network=172.16.1.8/30

/ip firewall address-list
add address=172.16.1.0/24 list=IP-INTERNAL
add address=192.168.1.0/24 list=IP-CLIENT
add address=192.168.2.0/24 list=IP-CLIENT
add address=192.168.3.0/24 list=IP-CLIENT
add address=192.168.4.0/24 list=IP-CLIENT
/ip firewall filter
add action=drop chain=input dst-address-list=IP-INTERNAL protocol=icmp src-address-list=IP-CLIENT
```

## Script R-ST-01
```
Konfigurasi VLAN 100,101,102 dan 103
Konfigurasi Trunking ke SW-ACC-01
Konfigurasi dhcp server
Konfigurasi Default gateway
Konfigurasi loadbalancing PCC dan failover
Konfigurasi Network Address Translation
Konfigurasi hotspot untuk guest dan karyawan
Konfigurasi firewall dengan ketentuan VLAN 100 dapat melakukan ping ke VLAN 101 akan tetapi VLAN 101 tidak bisa melakukan ping ke VLAN 100

BW:
VLAN 100 =  5 Mb

VLAN 101 =  .iso file = 1Mb
            .pdf file = 2Mb
Browsing =  5Mb

VLAN 102 =  .iso file = 1Mb
            .pdf file = 2Mb
Browsing =  5Mb

VLAN 103 =  2 Mb
```

## Script SW-ACC-01
```
Konfigurasi trunking ke router R-ST-01
Konfigruasi trunking ke AP
Konfigurasi VLAN 100 dan VLAN 101
```

## Script AP-ACC-01
```
Konfigurasi Virtual Access Point
Konfigurasi Access Point mode bridge
Konfigurasi authentication access untuk SSID karyawan
```
