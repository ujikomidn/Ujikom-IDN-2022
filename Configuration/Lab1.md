# Lab 1

Konfigurasi untuk Lab 1 Ujikom 

By: Andiama

## Table of Contents
- [Config R1](#config-r1)
- [Config R2](#config-r2)
- [Config VPCS Kiri](#config-vpcs-kiri)
- [Config VPCS Kanan](#config-vpcs-kanan)

## Topology
![image](https://user-images.githubusercontent.com/100014814/159838210-fe70b156-40ec-4345-afe2-e158318b2d8a.png)

## Config R1
```
system identity set name=R1
ip dhcp-client add disabled=no interface=ether3
ip firewall nat add action=masquerade chain=srcnat out-interface=ether3
ip address add address=12.12.12.1/24 interface=ether2 network=12.12.12.0
ip address add address=192.168.10.1/24 interface=ether1 network=192.168.10.0
routing ospf area add area-id=0.0.0.1 name=area1
routing ospf instance set 0 distribute-default=always-as-type-2 redistribute-connected=as-type-1 redistribute-static=as-type-2 router-id=1.1.1.1
routing ospf network add area=area1 network=12.12.12.0/24
routing ospf network add area=backbone network=192.168.10.0/24
routing ospf virtual-link add neighbor-id=2.2.2.2 transit-area=area1
```

## Config R2
```
system identity set name=R2
ip address add address=12.12.12.2/24 interface=ether2 network=12.12.12.0
ip address add address=192.168.20.1/24 interface=ether1 network=192.168.20.0
routing ospf area add area-id=0.0.0.1 name=area1
routing ospf area add area-id=0.0.0.2 name=area2
routing ospf instance set 0 redistribute-connected=as-type-1 router-id=2.2.2.2
routing ospf network add area=area1 network=12.12.12.0/24
routing ospf network add area=area2 network=192.168.20.0/24
routing ospf virtual-link add neighbor-id=1.1.1.1 transit-area=area1
```

## Config VPCS Kiri
```
ip 192.168.10.11 255.255.255.0 192.168.10.1
```

## Config VPCS Kanan
```
ip 192.168.20.11 255.255.255.0 192.168.20.1
```
