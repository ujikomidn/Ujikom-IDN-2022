# Lab 2

Konfigurasi untuk Lab 2 Ujikom

By: A Sunnah

## Topology
![image](https://user-images.githubusercontent.com/100014814/160052610-0a4959cd-4a63-4b8a-9f7f-71188555481c.png)

## Config R1
```
/system identity
set name=A-SUNNAH-R1

/interface bonding
add mode=802.3ad name=LACP-R1-3 slaves=ether2,ether3

/ip address
add address=12.12.12.1/24 interface=ether1 network=12.12.12.0
add address=13.13.13.1/24 interface=LACP-R1-3 network=13.13.13.0
add address=192.168.10.1/24 interface=ether4 network=192.168.10.0

/routing ospf network
add area=backbone network=12.12.12.0/24
add area=backbone network=13.13.13.0/24
add area=backbone network=192.168.10.0/24

/ip route
add gateway=13.13.13.3
```

## Config R2
```
/system identity
set name=A-SUNNAH-R2

/ip dhcp-client
add disabled=no interface=ether1

/ip address
add address=12.12.12.2/24 interface=ether2 network=12.12.12.0
add address=23.23.23.2/24 interface=ether3 network=23.23.23.0

/ip firewall nat
add action=masquerade chain=srcnat out-interface=ether1
/ip firewall filter
add action=drop chain=forward dst-address=103.28.12.165

/routing ospf network
add area=backbone network=12.12.12.0/24
add area=backbone network=23.23.23.0/24
```

## Config R3
```
/system identity
set name=A-SUNNAH-R3

/interface bonding
add mode=802.3ad name=LACP-R1-3 slaves=ether2,ether3

/ip address
add address=23.23.23.3/24 interface=ether1 network=23.23.23.0
add address=13.13.13.3/24 interface=LACP-R1-3 network=13.13.13.0
add address=192.168.20.1/24 interface=ether4 network=192.168.20.0

/routing ospf network
add area=backbone network=23.23.23.0/24
add area=backbone network=13.13.13.0/24
add area=backbone network=192.168.20.0/24

/ip route
add gateway=23.23.23.3
```

[NEXT](https://github.com/ujikomidn/Ujikom-IDN-2022/blob/main/Configuration/Lab3.md)
