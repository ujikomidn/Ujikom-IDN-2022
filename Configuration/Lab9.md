# Lab 9

Konfigurasi untuk Lab 9 Ujikom

By: Andiama

## Table of Contents
- [Config R1](#config-r1)
- [Config R2](#config-r2)
- [Config VPCS Kiri](#config-vpcs-kiri)
- [Config VPCS Kanan](#config-vpcs-kanan)

## Topology
![image](https://user-images.githubusercontent.com/100014814/160050060-95c41ba6-1536-4970-92fe-22538712ab43.png)

## Config R1
```
system identity set name=R1
ip dhcp-client add interface=ether1
ip address add address=192.168.10.1/24 interface=ether2 network=192.168.10.0
ip firewall nat add action=masquerade chain=srcnat out-interface=ether1
interface eoip add name=tunnel-eoip local-address=192.168.111.148 remote-address=192.168.111.149 tunnel-id=1
interface bridge add name=bridge1
interface bridge port add interface=ether2 bridge=bridge1
interface bridge port add interface=tunnel-eoip bridge=bridge1
```

## Config R2
```
system identity set name=R2
ip dhcp-client add interface=ether1
ip address add address=192.168.10.2/24 interface=ether2 network=192.168.10.0
ip firewall nat add action=masquerade chain=srcnat out-interface=ether1
interface eoip add name=tunnel-eoip local-address=192.168.111.149 remote-address=192.168.111.148 tunnel-id=1
interface bridge add name=bridge1
interface bridge port add interface=ether2 bridge=bridge1
interface bridge port add interface=tunnel-eoip bridge=bridge1
```

## Config VPCS Kiri
```
ip 192.168.10.3 255.255.255.0 192.168.10.1
```

## Config VPCS Kanan
```
ip 192.168.10.4 255.255.255.0 192.168.10.2
```
