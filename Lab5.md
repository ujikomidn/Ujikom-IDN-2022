# Lab 6

Konfigurasi untuk Lab 6 Ujikom

## Script R1

```host R1
int e0/0
ip add 12.12.12.1 255.255.255.0
no sh
int e0/1
ip add 13.13.13.1 255.255.255.0
no sh
int l0
ip add 1.1.1.1 255.255.255.255

router ei 10
net 12.12.12.1 0.0.0.0
net 13.13.13.1 0.0.0.0
net 1.1.1.1 0.0.0.0

key ch EIGRP
key 1
key-s CCNP
int e0/0
ip authe mode eigrp 10 md5
ip authe key eigrp 10 EIGRP
int e0/1
ip authe mode eigrp 10 md5
ip authe key eigrp 10 EIGRP
delay 8008
```

## Script R2
```host R2
int e0/0
ip add 12.12.12.2 255.255.255.0
no sh
int e0/1
ip add 23.23.23.2 255.255.255.0
no sh
int l0
ip add 2.2.2.2 255.255.255.255

router ei 10
net 12.12.12.2 0.0.0.0
net 23.23.23.2 0.0.0.0
net 2.2.2.2 0.0.0.0

key ch EIGRP
key 1
key-s CCNP
int e0/0
ip authe mode eigrp 10 md5
ip authe key eigrp 10 EIGRP
int e0/1
ip authe mode eigrp 10 md5
ip authe key eigrp 10 EIGRP
```

## Script R3

```host R3
int e0/0
ip add 23.23.23.3 255.255.255.0
no sh
int e0/1
ip add 13.13.13.3 255.255.255.0
no sh
int l0
ip add 3.3.3.3 255.255.255.255

router ei 10
net 13.13.13.3 0.0.0.0
net 23.23.23.3 0.0.0.0
net 3.3.3.3 0.0.0.0

key ch EIGRP
key 1
key-s CCNP
int e0/0
ip authe mode eigrp 10 md5
ip authe key eigrp 10 EIGRP
int e0/1
ip authe mode eigrp 10 md5
ip authe key eigrp 10 EIGRP
```
