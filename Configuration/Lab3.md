# Lab 3

Konfigurasi untuk Lab 3 Ujikom 

By: A Sunnah
## Table of Contents
- [Config R-1](#config-r-1)
- [Config R-2](#config-r-2)
- [Config R-3](#config-r-3)
- [Config R-4](#config-r-4)

## Topology
![image](https://user-images.githubusercontent.com/100014814/159838915-1f0700c2-ef48-4dd6-afd5-a0f8f6437579.png)

## Config R-1
```
hostname A-Sunnah-R01
interface Loopback0
ip address 1.1.1.1 255.255.255.255
interface Loopback1
ip address 11.11.11.11 255.255.255.255
interface Ethernet0/1
no shutdown
ip address 12.12.12.1 255.255.255.0
interface Ethernet0/0
no shutdown
ip address 14.14.14.1 255.255.255.0

router eigrp 12
network 1.1.1.1 0.0.0.0
network 12.12.12.0 0.0.0.255

key chain a-sunnah
key 1
key-string eigrp
interface Ethernet0/1
ip authentication mode eigrp 12 md5
ip authentication key-chain eigrp 12 a-sunnah

router bgp 123
network 14.14.14.0 mask 255.255.255.0
network 11.11.11.11 mask 255.255.255.255
neighbor 2.2.2.2 remote-as 123
neighbor 2.2.2.2 update-source loopback 0
neighbor 14.14.14.4 remote-as 4.1
```

## Config R-2
```
hostname A-Sunnah-R02
interface Loopback0
ip address 2.2.2.2 255.255.255.255
interface Ethernet0/0
ip address 12.12.12.2 255.255.255.0
no shutdown
interface Ethernet0/1
ip address 23.23.23.2 255.255.255.0
no shutdown

router eigrp 12
network 12.12.12.0 0.0.0.255
network 2.2.2.2 0.0.0.0
redistribute ospf 23 metric 1 1 1 1 1

router ospf 23
network 23.23.23.0 0.0.0.255 area 0
redistribute eigrp 12 subnets

key chain a-sunnah
key 1
key-string eigrp
interface Ethernet0/0
ip authentication mode eigrp 12 md5
ip authentication key-chain eigrp 12 a-sunnah
interface Ethernet0/1
ip ospf authentication message-digest
ip ospf message-digest-key 1 md5 a-sunnah

router bgp 123
neighbor 1.1.1.1 remote-as 123
neighbor 1.1.1.1 route-reflector-client
neighbor 1.1.1.1 update-source loopback 0
neighbor 3.3.3.3 remote-as 123
neighbor 3.3.3.3 route-reflector-client
neighbor 3.3.3.3 update-source loopback 0
```

## Config R-3
```
hostname A-Sunnah-R03
interface Loopback0
ip address 3.3.3.3 255.255.255.255
interface Loopback1
ip address 33.33.33.33 255.255.255.255
interface Ethernet0/0
no shutdown
ip address 34.34.34.3 255.255.255.0
interface Ethernet0/1
no shutdown
ip address 23.23.23.3 255.255.255.0

router ospf 23
network 3.3.3.3 0.0.0.0 area 0
network 23.23.23.0 0.0.0.255 area 0

interface Ethernet0/1
ip ospf authentication message-digest
ip ospf message-digest-key 1 md5 a-sunnah

router bgp 123
network 34.34.34.0 mask 255.255.255.0
network 33.33.33.33 mask 255.255.255.255
neighbor 2.2.2.2 remote-as 123
neighbor 2.2.2.2 update-source loopback 0
neighbor 34.34.34.4 remote-as 4.1
```


## Config R-4
```
hostname A-Sunnah-R01
interface Loopback0
ip address 4.4.4.4 255.255.255.255
interface Ethernet0/0
no shutdown
ip address 34.34.34.4 255.255.255.0
interface Ethernet0/1
no shutdown
ip address 14.14.14.4 255.255.255.0

router bgp 4.1
network 4.4.4.4 mask 255.255.255.255
network 14.14.14.0 mask 255.255.255.0
network 34.34.34.0 mask 255.255.255.0
neighbor 14.14.14.1 remote-as 123
neighbor 34.34.34.3 remote-as 123
```

[NEXT](https://github.com/ujikomidn/Ujikom-IDN-2022/blob/main/Configuration/Lab4.md)
