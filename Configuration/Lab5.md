# Lab 5

Konfigurasi untuk Lab 5 Ujikom

By: Andiama

## Table of Contents
- [Config R1](#config-r1)
- [Config R2](#config-r2)
- [Config R3](#config-r3)

## Topology
![image](https://user-images.githubusercontent.com/100014814/159839480-9b13058e-b091-4a56-855f-5ec3f8531103.png)

## Config R1
```
host R1
interface e0/0
ip address 12.12.12.1 255.255.255.0
no shutdown
interface e0/1
ip address 13.13.13.1 255.255.255.0
no shutdown
interface l0
ip address 1.1.1.1 255.255.255.255

router eigrp 10
network 12.12.12.1 0.0.0.0
network 13.13.13.1 0.0.0.0
network 1.1.1.1 0.0.0.0

key chain EIGRP
key 1
key-string CCNP
interface e0/0
ip authentication mode eigrp 10 md5
ip authentication key eigrp 10 EIGRP
interface e0/1
ip authentication mode eigrp 10 md5
ip authentication key eigrp 10 EIGRP
delay 8008135
```

## Config R2
```
host R2
interface e0/0
ip address 12.12.12.2 255.255.255.0
no shutdown
interface e0/1
ip address 23.23.23.2 255.255.255.0
no shutdown
interface l0
ip address 2.2.2.2 255.255.255.255

router eigrp 10
network 12.12.12.2 0.0.0.0
network 23.23.23.2 0.0.0.0
network 2.2.2.2 0.0.0.0

key chain EIGRP
key 1
key-string CCNP
interface e0/0
ip authentication mode eigrp 10 md5
ip authentication key eigrp 10 EIGRP
interface e0/1
ip authentication mode eigrp 10 md5
ip authentication key eigrp 10 EIGRP
```

## Config R3
```
host R3
interface e0/0
ip address 23.23.23.3 255.255.255.0
no shutdown
interface e0/1
ip address 13.13.13.3 255.255.255.0
no shutdown
interface l0
ip address 3.3.3.3 255.255.255.255

router eigrp 10
network 13.13.13.3 0.0.0.0
network 23.23.23.3 0.0.0.0
network 3.3.3.3 0.0.0.0

key chain EIGRP
key 1
key-string CCNP
interface e0/0
ip authentication mode eigrp 10 md5
ip authentication key eigrp 10 EIGRP
interface e0/1
ip authentication mode eigrp 10 md5
ip authentication key eigrp 10 EIGRP
```
