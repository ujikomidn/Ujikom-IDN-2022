# Lab 6

Konfigurasi untuk Lab 6 Ujikom

By: A Sunnah & Andiama

## Topology
![image](https://user-images.githubusercontent.com/100014814/159840612-e94732e6-6b82-45a7-86e4-cb553819d78e.png)

## Config Router
```
system identity set name=idnrouter
ip dhcp-client add disabled=no interface=wlan1
ip firewall nat add action=masquerade chain=srcnat out-interface=wlan1
ip address add address=172.16.10.1/24 interface=ether2 network=172.16.10.0
ip address add address=172.16.20.1/24 interface=ether3 network=172.16.20.0
ip pool add name=dhcp_pool0 ranges=172.16.10.11-172.16.10.254
ip pool add name=dhcp_pool1 ranges=172.16.20.11-172.16.20.254
ip dhcp-server network add address=172.16.10.0/24 dns-server=8.8.8.8 gateway=172.16.10.1
ip dhcp-server network add address=172.16.20.0/24 dns-server=8.8.8.8 gateway=172.16.20.1
ip dhcp-server add address-pool=dhcp_pool0 disabled=no interface=ether2 name=dhcp1
ip dhcp-server add address-pool=dhcp_pool1 disabled=no interface=ether3 name=dhcp2
ip hotspot add address-pool=dhcp_pool0 disabled=no interface=ether2 name=hotspot1 profile=hsprof1
ip hotspot add address-pool=dhcp_pool1 disabled=no interface=ether3 name=hotspot2 profile=hsprof2
ip hotspot profile add dns-name=ujikom.net hotspot-address=172.16.10.1 name=hsprof1
ip hotspot profile add dns-name=ujikom.net hotspot-address=172.16.20.1 name=hsprof2
ip hotspot user add name=farros password=keceparah
tool mac-server mac-winbox set allowed-interface-list=none
ip firewall filter add chain=input  src-address=!172.16.20.254 protocol=tcp dst-port=8291 action=drop
ip firewall filter add chain=forward dst-address=119.81.42.42 action=drop
```

[NEXT](https://github.com/ujikomidn/Ujikom-IDN-2022/blob/main/Configuration/Lab7.md)
