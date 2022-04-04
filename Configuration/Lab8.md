# Lab 8
Konfigurasi untuk Lab 8 Ujikom

By: Andiama

## Table of Contents
- [Mending ke ******https://www.server-world.info/en/note?os=Windows_Server_2019&p=download****** soalnya lebih shahih](#mending-ke-------https---wwwserver-worldinfo-en-note-os-windows-server-2019-p-download-------soalnya-lebih-shahih)
- [Server Configuration](#server-configuration)
  * [Initial Configuration](#initial-configuration)
    + [Change Administrator Name](#change-administrator-name)
    + [Change Computer Name](#change-computer-name)
    + [Set DNS Name](#set-dns-name)
    + [Set Static IP Address](#set-static-ip-address)
  * [Install DNS Server](#install-dns-server)
  * [Install DHCP Server](#install-dhcp-server)

## Topology
![image](https://user-images.githubusercontent.com/100014814/160049895-6f7f0696-4831-49f7-bfd2-906fcc04538d.png)


## Mending ke ******https://www.server-world.info/en/note?os=Windows_Server_2019&p=download****** soalnya lebih shahih

## Server Configuration

### Initial Configuration

#### Change Administrator Name
```
PS C:\Users\Administrator> Rename-LocalUser -Name "Administrator" -NewName "AdminIDN" 
```
#### Change Computer Name
```
PS C:\Users\AdminIDN> Rename-Computer -NewName Ujikom -Force -PassThru 
```
#### Set DNS Name
```
PS C:\Users\AdminIDN> Set-ItemProperty "HKLM:\SYSTEM\CurrentControlSet\Services\Tcpip\Parameters\" –Name "NV Domain" –Value "idn.id" -PassThru 
PS C:\Users\AdminIDN> Restart-Computer -Force
```
#### Set Static IP Address
Pertama, cek status Ethernet pada Server
```
PS C:\Users\AdminIDN> Get-NetIPInterface -AddressFamily IPv4 

ifIndex InterfaceAlias                  AddressFamily NlMtu(Bytes) InterfaceMetric Dhcp     ConnectionState PolicyStore
------- --------------                  ------------- ------------ --------------- ----     --------------- -----------
6       Ethernet                        IPv4                  1500              15 Enabled  Connected       ActiveStore
1       Loopback Pseudo-Interface 1     IPv4            4294967295              75 Disabled Connected       ActiveStore

```

Jika DHCPnya menyala, matikan terlebih dahulu
```
PS C:\Users\AdminIDN> Set-NetIPInterface -InterfaceIndex 6 -Dhcp Disabled
```

Kemudian set IP Address sesuai keinginan
```
PS C:\Users\AdminIDN> New-NetIPAddress -InterfaceIndex 6 -AddressFamily IPv4 -IPAddress "192.168.10.11" -PrefixLength 24 -DefaultGateway "192.168.10.10" 
```
Dan set DNS Server
```
PS C:\Users\Administrator> Set-DnsClientServerAddress -InterfaceIndex 6 -ServerAddresses "192.168.10.111" -PassThru 
```
### Install DNS Server
Install DNS dan DHCP Server agar tidak restart server 2 kali (jangan lupa Run as Administrator)
```
PS C:\Users\AdminIDN> Install-WindowsFeature DNS -IncludeManagementTools 
PS C:\Users\AdminIDN> Install-WindowsFeature DHCP -IncludeManagementTools
PS C:\Users\AdminIDN> Restart-Computer -Force 
```

Tambahkan Forward dan Reverse Lookup Zone
```
PS C:\Users\AdminIDN> Add-DnsServerPrimaryZone -Name "idn.id" -ZoneFile "idn.id.dns" -DynamicUpdate None -PassThru
PS C:\Users\AdminIDN> Add-DnsServerPrimaryZone -NetworkID 192.0.0.0/24 -ZoneFile "0.0.192.in-addr.arpa.dns" -DynamicUpdate None -PassThru 
```

Kemudian tambahkan Record untuk A/PTR
```
PS C:\Users\AdminIDN> Add-DnsServerResourceRecordA -Name "ujikomtest" -ZoneName "idn.id" -IPv4Address "192.168.10.101" -TimeToLive 01:00:00 -CreatePtr -PassThru 
```

Dan terakhir, coba tes apakah DNS Server sudah terbaca
```
PS C:\Users\AdminIDN> nslookup ujikomtest.idn.id localhost 
Server:  UnKnown
Address:  ::1

Name:    ujikomtest.idn.id
Address:  192.168.10.101

PS C:\Users\Administrator> nslookup 192.168.10.101 localhost 
Server:  UnKnown
Address:  ::1

Name:    ujikomtest.idn.id
Address:  192.168.10.101
```
### Install DHCP Server

Server akan restart setelah memasukkan command diatas. Selanjutnya, setting agar DHCP Server bisa dipakai oleh PC.
```
PS C:\Users\AdminIDN> Add-DhcpServerSecurityGroup -ComputerName "Ujikom.idn.id" 
PS C:\Users\AdminIDN> Add-DhcpServerv4Scope -Name "Ujikom" `
-StartRange 192.168.10.112 `
-EndRange 192.168.10.254 `
-SubnetMask 255.255.255.0 `
-LeaseDuration 8.00:00:00 `
-State Active `
-PassThru 
PS C:\Users\AdminIDN> Set-DhcpServerv4OptionValue -DnsDomain "idn.id" `
-DnsServer "192.168.10.101" `
-Router "192.168.10.1" `
-ScopeId "192.168.10.0" `
-PassThru 
PS C:\Users\AdminIDN> Restart-Service DHCPServer 
```
[NEXT](https://github.com/ujikomidn/Ujikom-IDN-2022/blob/main/Configuration/Lab9.md)
