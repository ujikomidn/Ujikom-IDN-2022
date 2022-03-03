# Lab 3

Konfigurasi untuk Lab 3 Ujikom 

By: A Sunnah

## Script R-1
```
hostname A-Sunnah-R01
interface Loopback0
ip address 1.1.1.1 255.255.255.255
interface Ethernet0/0
no shutdown
ip address 12.12.12.1 255.255.255.0
interface Ethernet0/1
no shutdown
ip address 14.14.14.1 255.255.255.0

router eigrp 10
network 1.1.1.1 0.0.0.0
network 12.12.12.0 0.0.0.255

router bgp 123
network 14.14.14.0 mask 255.255.255.0
neighbor 2.2.2.2 remote-as 123
neighbor 14.14.14.4 remote-as 262145

key chain fkm
key 1
key-string eigrp
interface Ethernet0/0
ip authentication mode eigrp 10 md5
ip authentication key-chain eigrp 10 fkm
```

## Script R-2
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

router eigrp 10
network 12.12.12.0 0.0.0.255
redistribute ospf 10 metric 1 1 1 1 1

router ospf 10
redistribute eigrp 10 subnets
network 2.2.2.2 0.0.0.0 area 0
network 23.23.23.0 0.0.0.255 area 0

router bgp 123
neighbor 12.12.12.1 remote-as 123
neighbor 12.12.12.1 route-reflector-client
neighbor 23.23.23.3 remote-as 123
neighbor 23.23.23.3 route-reflector-client

key chain fkm
key 1
key-string eigrp
interface Ethernet0/0
ip authentication mode eigrp 10 md5
ip authentication key-chain eigrp 10 fkm
interface Ethernet0/1
ip ospf authentication message-digest
ip ospf message-digest-key 1 md5 fkm
```

## Script R-3
```
hostname A-Sunnah-R03
interface Loopback0
ip address 3.3.3.3 255.255.255.255
interface Ethernet0/0
no shutdown
ip address 34.34.34.3 255.255.255.0
interface Ethernet0/1
no shutdown
ip address 23.23.23.3 255.255.255.0

router ospf 10
network 3.3.3.3 0.0.0.0 area 0
network 23.23.23.0 0.0.0.255 area 0

router bgp 123
network 34.34.34.0 mask 255.255.255.0
neighbor 2.2.2.2 remote-as 123
neighbor 34.34.34.4 remote-as 262145

interface Ethernet0/1
ip ospf authentication message-digest
ip ospf message-digest-key 1 md5 fkm
```


## Script R-4
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

router bgp 262145
network 4.4.4.4 mask 255.255.255.255
network 14.14.14.0 mask 255.255.255.0
network 34.34.34.0 mask 255.255.255.0
neighbor 14.14.14.1 remote-as 123
neighbor 34.34.34.3 remote-as 123
```
