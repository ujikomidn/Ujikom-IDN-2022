# Lab 9

Konfigurasi untuk Lab 9 Ujikom

By: A Sunnah

## Table of Contents
- [Config R1](#config-r1)
- [Config R2](#config-r2)
- [Config VPCS Kiri](#config-vpcs-kiri)
- [Config VPCS Kanan](#config-vpcs-kanan)

## Config R1
```
system identity set name=R1
ip dhcp-client add interface=ether1
ip address add address=192.168.10.1/24 interface=ether2
ip firewall nat add action=masquerade chain=srcnat out-interface=ether1
interface gre add name=GRE-SITE-A local-address=192.168.111.148 remote-address=192.168.111.149
ip address add address=10.10.10.1/30 interface=GRE-SITE-A
ip route add dst-address=192.168.10.0/24 gateway=10.10.10.1
```

## Config R2
```
system identity set name=R2
ip dhcp-client add interface=ether1
ip address add address=192.168.20.2/24 interface=ether2
ip firewall nat add action=masquerade chain=srcnat out-interface=ether1
interface gre add name=GRE-SITE-B local-address=192.168.111.149 remote-address=192.168.111.148
ip address add address=10.10.10.2/30 interface=GRE-SITE-B
ip route add dst-address=192.168.20.0/24 gateway=10.10.10.2
```

## Config VPCS Kiri
```
ip 192.168.10.3 255.255.255.0 192.168.10.1
```

## Config VPCS Kanan
```
ip 192.168.10.4 255.255.255.0 192.168.10.2
```
