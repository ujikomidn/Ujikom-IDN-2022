# Lab 2

Konfigurasi untuk Lab 2 Ujikom

By: Andiama

## Config Router
```
system identity set name=idnrouter
ip dhcp-client add disabled=no interface=wlan1
ip firewall nat add action=masquerade chain=srcnat out-interface=wlan1
ip address add address=192.168.10.1/24 interface=ether2 network=192.168.10.0
ip address add address=192.168.20.1/24 interface=ether3 network=192.168.20.0
ip pool add name=dhcp_pool0 ranges=192.168.10.2-192.168.10.254
ip pool add name=dhcp_pool1 ranges=192.168.20.2-192.168.20.254
ip dhcp-server network add address=192.168.10.0/24 dns-server=8.8.8.8,1.1.1.1 gateway=192.168.10.1
ip dhcp-server network add address=192.168.20.0/24 dns-server=8.8.8.8,1.1.1.1 gateway=192.168.20.1
ip dhcp-server add address-pool=dhcp_pool0 disabled=no interface=ether2 name=dhcp1
ip dhcp-server add address-pool=dhcp_pool1 disabled=no interface=ether3 name=dhcp2
ip hotspot add address-pool=dhcp_pool0 disabled=no interface=ether2 name=hotspot1 profile=hsprof1
ip hotspot add address-pool=dhcp_pool1 disabled=no interface=ether3 name=hotspot2 profile=hsprof2
ip hotspot profile add dns-name=ujikom.net hotspot-address=192.168.10.1 name=hsprof1
ip hotspot profile add dns-name=ujikom.net hotspot-address=192.168.20.1 name=hsprof2
ip hotspot user add name=farros password=keceparah
```
