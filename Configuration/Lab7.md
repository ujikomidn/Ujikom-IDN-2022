# Lab 7

Konfigurasi untuk Lab 7 Ujikom

By: Andiama

## Table of Contents
- [Cacti Installation](#cacti-installation)
  * [LAMP Server Installation](#lamp-server-installation)
  * [MariaDB Installation](#mariadb-installation)
  * [Cacti Installation and Configuration](#cacti-installation-and-configuration)
  * [Apache Configuration for Cacti](#apache-configuration-for-cacti)
  * [Cacti Frontend Installation](#cacti-frontend-installation)
- [SNMP on Mikrotik](#snmp-on-mikrotik)
- [Cacti SNMP Setup](#cacti-snmp-setup)
  * [Script CPU Usage](#script-cpu-usage)
  * [Script Memory Usage](#script-memory-usage)

## Topology
![image](https://user-images.githubusercontent.com/100014814/161365981-bf19df01-07c6-4b85-9bd1-b5abc04e973b.png)

## Cacti Installation

### LAMP Server Installation
LAMP merupakan singkatan dari Linux, Apache, MariaDB dan PHP. Mari kita install semuanya dengan 2 command.
```
sudo apt update && sudo apt upgrade
sudo apt install apache2 mariadb-server mariadb-client libmariadb-dev php php-mysql php-mysqlnd php-ldap php-bcmath php-mbstring php-gd php-pdo php-xml libapache2-mod-php php-gmp php-snmp 
```

Selanjutnya, ubah beberapa hal dalam "php.ini". Kalian bisa menggunakan F6 ataupun CTRL + W untuk mencari line yang diperlukan
```
sudo nano /etc/php/7.4/apache2/php.ini
>>>post_max_size = 16M
>>>upload_max_filesize = 2M
>>>max_execution_time = 300
>>>max_input_time = 300
>>>memory_limit = 512M
>>>session.auto_start = 0
>>>mbstring.func_overload = 0 (ilangin ; nya)
>>>date.timezone = Asia/Jakarta (ini juga ilangin ; nya)
>>>extension=php_gmp (kayanya ga perlu)

sudo nano /etc/php/7.4/cli/php.ini
>>>max_execution_time = 0
>>>date.timezone = Asia/Jakarta (jangan lupa ilangin ; nya)
```

Kemudian restart daemon dari apache2 agar apache2 mendetect perubahan pada PHP
```
sudo systemctl restart apache2.service	
```

### MariaDB Installation

Karena kita sudah menginstall MariaDB sebelumnya, kita tinggal perlu run daemon untuk MariaDB lalu install menggunakan "mysql_secure_installation".

Ikutin aja kayak dibawah, insyaallah work 100% no hack no root indo sub kalo ikutin step by stepnya

*(BTW inget password root ya, jangan sampe lupa, simpen di notepad kalo misalnya kalian menderita demensia ato apalah itu)*
```
aidan@ujikommunication:~$ sudo systemctl stop mysql
aidan@ujikommunication:~$ sudo usermod -d /var/lib/mysql/ mysql
aidan@ujikommunication:~$ sudo systemctl start mysql
aidan@ujikommunication:~$ sudo mysql_secure_installation

Enter current password for root (enter for none): <masukkan password>

You already have a root password set, so you can safely answer 'n'.

Change the root password? [Y/n] n
 ... skipping.

Remove anonymous users? [Y/n] y
 ... Success!

Disallow root login remotely? [Y/n] n
 ... skipping.

Remove test database and access to it? [Y/n] y
 - Dropping test database...
 ... Success!
 - Removing privileges on test database...
 ... Success!

Reload privilege tables now? [Y/n] y
 ... Success!

Cleaning up...

All done!  If you've completed all of the above steps, your MariaDB
installation should now be secure.

Thanks for using MariaDB!
```

Next, buat database, user dan password untuk MariaDB kita.
```
sudo mysql -u root -p
MariaDB [(none)]> create database idncacti character set utf8mb4 collate utf8mb4_unicode_ci;
MariaDB [(none)]> grant all on idncacti.* to usermantab@localhost identified by 'farrosjoss';
MariaDB [(none)]> flush privileges;
MariaDB [(none)]> exit
```

Edit konfigurasi MariaDB agar bisa digunakan oleh Cacti
```
sudo nano /etc/mysql/mariadb.conf.d/50-server.cnf
************Masukkan konfigurasi dibawah ke paling bawah text************
character-set-server            = utf8mb4
collation-server                = utf8mb4_unicode_ci
max_heap_table_size             = 128M
tmp_table_size                  = 64M
join_buffer_size                = 128M
innodb_file_format              = Barracuda
innodb_large_prefix             = 1
innodb_buffer_pool_size         = 1G
innodb_flush_log_at_timeout     = 3
innodb_read_io_threads          = 32
innodb_write_io_threads         = 16
innodb_io_capacity              = 5000
innodb_io_capacity_max          = 10000
innodb_doublewrite              = OFF
innodb_buffer_pool_instances    = 11
*************************************************************************
sudo systemctl restart mariadb
```

Lalu masukkan Data untuk Time Zone ke dalam MySQL
```
sudo mysql mysql < /usr/share/mysql/mysql_test_data_timezone.sql  
sudo mysql
MariaDB [(none)]> grant select on mysql.time_zone_name to usermantab@localhost;
MariaDB [(none)]> flush privileges;
MariaDB [(none)]> exit
```

### Cacti Installation and Configuration
Selanjutnya, install Cacti beserta dependencynya
```
sudo apt install rrdtool snmp snmpd snmp-mibs-downloader libsnmp-dev
sudo wget https://www.cacti.net/downloads/cacti-latest.tar.gz
sudo mkdir /var/www/html/cacti
sudo tar xzf cacti-latest.tar.gz -C /var/www/html/cacti
sudo mv /var/www/html/cacti/cacti-1.2.19/* /var/www/html/cacti/
sudo mysql idncacti < /var/www/html/cacti/cacti.sql
sudo chown -R www-data:www-data /var/www/html/cacti/
sudo chmod -R 775 /var/www/html/cacti/
```
Setelah itu, konfigurasikan agar Cacti menggunakan database yang telah kita berikan
```
sudo nano /var/www/html/cacti/include/config.php

********Edit data agar seperti dibawah ini********
$database_type     = 'mysql';
$database_default  = 'idncacti';
$database_hostname = 'localhost';
$database_username = 'usermantab';
$database_password = 'farrosjoss';
$database_port     = '3306';
```

Lalu buat log untuk Cacti dan buat Cron Job untuk Cacti
```
sudo touch /var/www/html/cacti/log/cacti.log
sudo chmod -R 775 /var/www/html/cacti/
sudo chown -R www-data:www-data /var/www/html/cacti/
sudo nano /etc/cron.d/cacti

********Tambahkan line seperti dibawah pada file tersebut********
*/5 * * * * www-data php /var/www/html/cacti/poller.php > /dev/null 2>&1
```
### Apache Configuration for Cacti
Kemudian konfigurasikan Apache agar dapat berjalan untuk Cacti
```
sudo nano /etc/apache2/sites-available/cacti.conf

********Tambahkan line dibawah ke dalam cacti.conf yang baru dibuat********
Alias /cacti    /var/www/html/cacti
<Directory /var/www/html/cacti/>
<IfModule mod_authz_core.c>
Require all granted
</IfModule>
</Directory>
```
Terakhir, test Apache untuk Cacti. Jika outputnya "Syntax OK", maka langsung restart daemon Apache
```
sudo apachectl configtest
sudo systemctl restart apache2
```
**Jika mendapat error "Could not reliably determine the server's fully qualified domain name, using 127.0.1.1. Set the 'ServerName' directive globally to suppress this message", lakukan seperti dibawah ini baru lakukan hal diatas*
```
sudo nano /etc/apache2/apache2.conf

********Tambahkan line dibawah ke bagian paling bawah konfigurasi Apache********
ServerName localhost
```

### Cacti Frontend Installation
Buka IP VM yang dimiliki lalu tambahkan "/cacti". Misalnya "192.168.111.143/cacti"

Masuk dengan kredensial default, yaitu "admin/admin"
![image](https://user-images.githubusercontent.com/100014814/157605745-7e5d16bc-55f4-4d6f-a8f1-b078c19eda2e.png)

Ubah password Cacti sesuka kalian, kali ini saya akan menggunakan "Farr0$joss"
![image](https://user-images.githubusercontent.com/100014814/157606509-af7aab35-163a-4551-a6af-88102ca129dd.png)

Centang agreementnya kemudian klik "Begin"
![image](https://user-images.githubusercontent.com/100014814/157606963-6a969d4c-4440-4210-9dc1-0f69854fe02a.png)

Klik "Next" jika dependency kalian sudah aman sentosa
![image](https://user-images.githubusercontent.com/100014814/157615339-ecad1d86-9606-4342-bb7e-1b7b257ff1fc.png)

Set Cacti sebagai Primary Server kemudian pastikan Database Local sudah sesuai dengan yang kita masukkan sebelumnya
![image](https://user-images.githubusercontent.com/100014814/157615626-f6d4b4eb-ff80-45a1-9cfe-20fd691b68d4.png)

Klik "Next" hingga melihat page ini. Jika sudah, klik "I have read this statement" kemudian klik "Next" lagi
![image](https://user-images.githubusercontent.com/100014814/157615936-4f8608a1-59db-4c5e-9ac0-86a8362d1d3f.png)

Lalu klik "Next" hingga bertemu page ini. Klik "Confirm Installation" lalu klik "Install"
![image](https://user-images.githubusercontent.com/100014814/157616397-effe9e21-f433-4d6f-8ebb-7125824c8e43.png)

Selesai menginstall, klik pada tombol "Get Started"
![image](https://user-images.githubusercontent.com/100014814/157775800-2c5fcfe3-196c-4586-88c3-2b2c57593b55.png)

Jika sudah masuk kedalam Cacti, kita akan langsung ke
**Configuration > Users**
![image](https://user-images.githubusercontent.com/100014814/157776159-106065ce-c5fe-44df-8dfa-fe505493a206.png)

Gantilah user admin dengan apapun yang diinginkan
![image](https://user-images.githubusercontent.com/100014814/157776476-65d4134b-db25-471e-90db-c38cde10838c.png)

## SNMP on Mikrotik
Pertama tama, tambahkan IP Address agar dapat di ping oleh PC dan Server
```
ip firewall nat add action=masquerade chain=srcnat out-interface=ether1
ip address add address=192.168.111.1/24 interface=ether2
ip address add address=192.168.10.1/24 interface=ether3
```
Kemudian tambahkan setting SNMP pada MikroTik
```
[admin@MikroTik] > ip pool add name=dhcp_pool0 ranges=192.168.111.1-192.168.111.254
[admin@MikroTik] > ip pool add name=dhcp_pool1 ranges=192.168.10.1-192.168.10.254
[admin@MikroTik] > ip dhcp-server network add address=192.168.111.0/24 dns-server=8.8.8.8 gateway=192.168.111.1
[admin@MikroTik] > ip dhcp-server network add address=192.168.10.0/24 dns-server=8.8.8.8 gateway=192.168.10.1
[admin@MikroTik] > ip dhcp-server add address-pool=dhcp_pool0 disabled=no interface=ether2 name=dhcp1
[admin@MikroTik] > ip dhcp-server add address-pool=dhcp_pool1 disabled=no interface=ether3 name=dhcp2
[admin@MikroTik] > snmp set enabled=yes
[admin@MikroTik] > snmp community set name=SNMPIDN 0
[admin@MikroTik] > snmp community print             
Flags: * - default, X - disabled 
 #    NAME     ADDRESSES                                      SECURITY   READ-ACCESS WRITE-ACCESS
 0 *  SNMPIDN  ::/0                                           none       yes         no  
[admin@MikroTik] > snmp print
          enabled: yes
          contact: 
         location: 
        engine-id: 
      trap-target: 
   trap-community: SNMPIDN
     trap-version: 1
  trap-generators: temp-exception
```
Lalu pada Server Linux, install dan coba SNMP
```
aidan@ujikommunication:~$ sudo apt-get install snmp
aidan@ujikommunication:~$ snmpwalk -v2c -c SNMPIDN 192.168.111.1
```

## Cacti SNMP Setup
Di server Cacti, buka
**Create > New Device**
![image](https://user-images.githubusercontent.com/100014814/158086957-9034d80a-6583-43a7-a20d-5b9ea6ab7a4b.png)

Masukkan Description, Hostname, Device Template, SNMP Version dan SNMP Community String. Jika sudah sama, klik "Create" pada pojok kanan bawah.
![image](https://user-images.githubusercontent.com/100014814/161364151-59464ee9-0f9d-4343-a2e3-75d2d9ba2135.png)

Klik "Create Graphs for this Device"
![image](https://user-images.githubusercontent.com/100014814/158502423-7bdb8f3b-2789-4f4b-8ee4-c815049ee101.png)

Kemudian refresh interface, baru set "In/Out Bits" untuk Interface yang mengarah ke Internet.
![image](https://user-images.githubusercontent.com/100014814/158503016-bcdb4e5f-7651-460d-832d-a4cf2ee6c138.png)

Selanjutnya, Klik "Import Templates" pada dropdown "Import/Export"
![image](https://user-images.githubusercontent.com/100014814/161364339-712e0f2a-3a71-4718-a83d-20f6b1c1bfc9.png)

Klik "Select a file" (jangan lupa matiin Preview Import Only), lalu import kode dibawah sebagai .xml (maksudnya masukin kode dibawah ke dalem .xml, namanya boleh apa aja) ((oiya sama jangan lupa pisahin script CPU sama Memorynya))

### Script CPU Usage
```
<cacti>	
	<hash_000005e698bfeed6840d045766635a9bacbbe0>
		<name>Mikrotik - CPU Usage</name>
		<graph>
			<t_title></t_title>
			<title>|host_description| - CPU Usage</title>
			<t_image_format_id></t_image_format_id>
			<image_format_id>1</image_format_id>
			<t_height></t_height>
			<height>120</height>
			<t_width></t_width>
			<width>500</width>
			<t_auto_scale></t_auto_scale>
			<auto_scale>on</auto_scale>
			<t_auto_scale_opts></t_auto_scale_opts>
			<auto_scale_opts>2</auto_scale_opts>
			<t_auto_scale_log></t_auto_scale_log>
			<auto_scale_log></auto_scale_log>
			<t_auto_scale_rigid></t_auto_scale_rigid>
			<auto_scale_rigid>on</auto_scale_rigid>
			<t_auto_padding></t_auto_padding>
			<auto_padding>on</auto_padding>
			<t_export></t_export>
			<export>on</export>
			<t_upper_limit></t_upper_limit>
			<upper_limit>100</upper_limit>
			<t_lower_limit></t_lower_limit>
			<lower_limit>0</lower_limit>
			<t_base_value></t_base_value>
			<base_value>1000</base_value>
			<t_unit_value></t_unit_value>
			<unit_value></unit_value>
			<t_unit_exponent_value></t_unit_exponent_value>
			<unit_exponent_value></unit_exponent_value>
			<t_vertical_label></t_vertical_label>
			<vertical_label>percent</vertical_label>
		</graph>
		<items>
			<hash_100005057d15afb7d8a06b90258c7988ee0b71>
				<task_item_id>hash_08000548f515755838763a71d68ed3d0fc7175</task_item_id>
				<color_id>FF0000</color_id>
				<graph_type_id>7</graph_type_id>
				<consolidation_function_id>1</consolidation_function_id>
				<cdef_id>0</cdef_id>
				<value></value>
				<gprint_id>hash_060005e9c43831e54eca8069317a2ce8c6f751</gprint_id>
				<text_format>CPU Usage</text_format>
				<hard_return></hard_return>
				<sequence>1</sequence>
			</hash_100005057d15afb7d8a06b90258c7988ee0b71>
			<hash_1000054b4bebaa5ec20a923a826fd072767e9a>
				<task_item_id>hash_08000548f515755838763a71d68ed3d0fc7175</task_item_id>
				<color_id>0</color_id>
				<graph_type_id>9</graph_type_id>
				<consolidation_function_id>4</consolidation_function_id>
				<cdef_id>0</cdef_id>
				<value></value>
				<gprint_id>hash_06000519414480d6897c8731c7dc6c5310653e</gprint_id>
				<text_format>Current:</text_format>
				<hard_return></hard_return>
				<sequence>2</sequence>
			</hash_1000054b4bebaa5ec20a923a826fd072767e9a>
			<hash_100005a49d96e10a3dc74512596d8485763649>
				<task_item_id>hash_08000548f515755838763a71d68ed3d0fc7175</task_item_id>
				<color_id>0</color_id>
				<graph_type_id>9</graph_type_id>
				<consolidation_function_id>1</consolidation_function_id>
				<cdef_id>0</cdef_id>
				<value></value>
				<gprint_id>hash_06000519414480d6897c8731c7dc6c5310653e</gprint_id>
				<text_format>Average:</text_format>
				<hard_return></hard_return>
				<sequence>3</sequence>
			</hash_100005a49d96e10a3dc74512596d8485763649>
			<hash_100005e7cb030d1036ffac6c675322240a4066>
				<task_item_id>hash_08000548f515755838763a71d68ed3d0fc7175</task_item_id>
				<color_id>0</color_id>
				<graph_type_id>9</graph_type_id>
				<consolidation_function_id>3</consolidation_function_id>
				<cdef_id>0</cdef_id>
				<value></value>
				<gprint_id>hash_06000519414480d6897c8731c7dc6c5310653e</gprint_id>
				<text_format>Maximum:</text_format>
				<hard_return></hard_return>
				<sequence>4</sequence>
			</hash_100005e7cb030d1036ffac6c675322240a4066>
		</items>
		<inputs>
			<hash_0900056e0f7a10eb1457ad6881bb3defc47140>
				<name>Data Source [cpu]</name>
				<description></description>
				<column_name>task_item_id</column_name>
				<items>hash_000005057d15afb7d8a06b90258c7988ee0b71|hash_0000054b4bebaa5ec20a923a826fd072767e9a|hash_000005a49d96e10a3dc74512596d8485763649|hash_000005e7cb030d1036ffac6c675322240a4066</items>
			</hash_0900056e0f7a10eb1457ad6881bb3defc47140>
		</inputs>
	</hash_000005e698bfeed6840d045766635a9bacbbe0>
	<hash_010005097a276485e6746cb677900ec4062d59>
		<name>Mikrotik - SNMP - CPU Load</name>
		<ds>
			<t_name></t_name>
			<name>|host_description| - Mikrotik - SNMP - CPU Load (5 min)</name>
			<data_input_id>hash_0300053eb92bb845b9660a7445cf9740726522</data_input_id>
			<t_rra_id></t_rra_id>
			<t_rrd_step></t_rrd_step>
			<rrd_step>300</rrd_step>
			<t_active></t_active>
			<active>on</active>
			<rra_items>hash_150005c21df5178e5c955013591239eb0afd46|hash_1500050d9c0af8b8acdc7807943937b3208e29|hash_1500056fc2d038fb42950138b0ce3e9874cc60|hash_150005e36f3adb9f152adfa5dc50fd2b23337e</rra_items>
		</ds>
		<items>
			<hash_08000548f515755838763a71d68ed3d0fc7175>
				<t_data_source_name></t_data_source_name>
				<data_source_name>cpu</data_source_name>
				<t_rrd_minimum></t_rrd_minimum>
				<rrd_minimum>0</rrd_minimum>
				<t_rrd_maximum></t_rrd_maximum>
				<rrd_maximum>100</rrd_maximum>
				<t_data_source_type_id></t_data_source_type_id>
				<data_source_type_id>1</data_source_type_id>
				<t_rrd_heartbeat></t_rrd_heartbeat>
				<rrd_heartbeat>600</rrd_heartbeat>
				<t_data_input_field_id></t_data_input_field_id>
				<data_input_field_id>0</data_input_field_id>
			</hash_08000548f515755838763a71d68ed3d0fc7175>
		</items>
		<data>
			<item_000>
				<data_input_field_id>hash_0700054276a5ec6e3fe33995129041b1909762</data_input_field_id>
				<t_value></t_value>
				<value>.1.3.6.1.2.1.25.3.3.1.2.1</value>
			</item_000>
			<item_001>
				<data_input_field_id>hash_070005012ccb1d3687d3edb29c002ea66e72da</data_input_field_id>
				<t_value></t_value>
				<value></value>
			</item_001>
			<item_002>
				<data_input_field_id>hash_0700059c55a74bd571b4f00a96fd4b793278c6</data_input_field_id>
				<t_value></t_value>
				<value></value>
			</item_002>
			<item_003>
				<data_input_field_id>hash_070005ad14ac90641aed388139f6ba86a2e48b</data_input_field_id>
				<t_value></t_value>
				<value></value>
			</item_003>
			<item_004>
				<data_input_field_id>hash_07000532285d5bf16e56c478f5e83f32cda9ef</data_input_field_id>
				<t_value></t_value>
				<value></value>
			</item_004>
			<item_005>
				<data_input_field_id>hash_07000592f5906c8dc0f964b41f4253df582c38</data_input_field_id>
				<t_value></t_value>
				<value></value>
			</item_005>
		</data>
	</hash_010005097a276485e6746cb677900ec4062d59>
	<hash_0300053eb92bb845b9660a7445cf9740726522>
		<name>Get SNMP Data</name>
		<type_id>2</type_id>
		<input_string></input_string>
		<fields>
			<hash_07000592f5906c8dc0f964b41f4253df582c38>
				<name>SNMP IP Address</name>
				<update_rra></update_rra>
				<regexp_match></regexp_match>
				<allow_nulls></allow_nulls>
				<type_code>hostname</type_code>
				<input_output>in</input_output>
				<data_name>management_ip</data_name>
			</hash_07000592f5906c8dc0f964b41f4253df582c38>
			<hash_07000532285d5bf16e56c478f5e83f32cda9ef>
				<name>SNMP Community</name>
				<update_rra></update_rra>
				<regexp_match></regexp_match>
				<allow_nulls></allow_nulls>
				<type_code>snmp_community</type_code>
				<input_output>in</input_output>
				<data_name>snmp_community</data_name>
			</hash_07000532285d5bf16e56c478f5e83f32cda9ef>
			<hash_070005ad14ac90641aed388139f6ba86a2e48b>
				<name>SNMP Username</name>
				<update_rra></update_rra>
				<regexp_match></regexp_match>
				<allow_nulls>on</allow_nulls>
				<type_code>snmp_username</type_code>
				<input_output>in</input_output>
				<data_name>snmp_username</data_name>
			</hash_070005ad14ac90641aed388139f6ba86a2e48b>
			<hash_0700059c55a74bd571b4f00a96fd4b793278c6>
				<name>SNMP Password</name>
				<update_rra></update_rra>
				<regexp_match></regexp_match>
				<allow_nulls>on</allow_nulls>
				<type_code>snmp_password</type_code>
				<input_output>in</input_output>
				<data_name>snmp_password</data_name>
			</hash_0700059c55a74bd571b4f00a96fd4b793278c6>
			<hash_070005012ccb1d3687d3edb29c002ea66e72da>
				<name>SNMP Version (1, 2, or 3)</name>
				<update_rra></update_rra>
				<regexp_match></regexp_match>
				<allow_nulls>on</allow_nulls>
				<type_code>snmp_version</type_code>
				<input_output>in</input_output>
				<data_name>snmp_version</data_name>
			</hash_070005012ccb1d3687d3edb29c002ea66e72da>
			<hash_0700054276a5ec6e3fe33995129041b1909762>
				<name>OID</name>
				<update_rra></update_rra>
				<regexp_match></regexp_match>
				<allow_nulls></allow_nulls>
				<type_code>snmp_oid</type_code>
				<input_output>in</input_output>
				<data_name>oid</data_name>
			</hash_0700054276a5ec6e3fe33995129041b1909762>
		</fields>
	</hash_0300053eb92bb845b9660a7445cf9740726522>
	<hash_150005c21df5178e5c955013591239eb0afd46>
		<name>Daily (5 Minute Average)</name>
		<x_files_factor>0.5</x_files_factor>
		<steps>1</steps>
		<rows>600</rows>
		<timespan>86400</timespan>
		<cf_items>1|3</cf_items>
	</hash_150005c21df5178e5c955013591239eb0afd46>
	<hash_1500050d9c0af8b8acdc7807943937b3208e29>
		<name>Weekly (30 Minute Average)</name>
		<x_files_factor>0.5</x_files_factor>
		<steps>6</steps>
		<rows>700</rows>
		<timespan>604800</timespan>
		<cf_items>1|3</cf_items>
	</hash_1500050d9c0af8b8acdc7807943937b3208e29>
	<hash_1500056fc2d038fb42950138b0ce3e9874cc60>
		<name>Monthly (2 Hour Average)</name>
		<x_files_factor>0.5</x_files_factor>
		<steps>24</steps>
		<rows>775</rows>
		<timespan>2678400</timespan>
		<cf_items>1|3</cf_items>
	</hash_1500056fc2d038fb42950138b0ce3e9874cc60>
	<hash_150005e36f3adb9f152adfa5dc50fd2b23337e>
		<name>Yearly (1 Day Average)</name>
		<x_files_factor>0.5</x_files_factor>
		<steps>288</steps>
		<rows>797</rows>
		<timespan>33053184</timespan>
		<cf_items>1|3</cf_items>
	</hash_150005e36f3adb9f152adfa5dc50fd2b23337e>
	<hash_060005e9c43831e54eca8069317a2ce8c6f751>
		<name>Normal</name>
		<gprint_text>%8.2lf %s</gprint_text>
	</hash_060005e9c43831e54eca8069317a2ce8c6f751>
	<hash_06000519414480d6897c8731c7dc6c5310653e>
		<name>Exact Numbers</name>
		<gprint_text>%8.0lf</gprint_text>
	</hash_06000519414480d6897c8731c7dc6c5310653e>
</cacti>
```

### Script Memory Usage
```
<cacti>	
	<hash_00000502d5e7579ab9b390756192bb43d3c1cd>
		<name>Mikrotik - Memory Usage</name>
		<graph>
			<t_title></t_title>
			<title>|host_description| - Memory Usage</title>
			<t_image_format_id></t_image_format_id>
			<image_format_id>1</image_format_id>
			<t_height></t_height>
			<height>120</height>
			<t_width></t_width>
			<width>500</width>
			<t_auto_scale></t_auto_scale>
			<auto_scale>on</auto_scale>
			<t_auto_scale_opts></t_auto_scale_opts>
			<auto_scale_opts>2</auto_scale_opts>
			<t_auto_scale_log></t_auto_scale_log>
			<auto_scale_log></auto_scale_log>
			<t_auto_scale_rigid></t_auto_scale_rigid>
			<auto_scale_rigid>on</auto_scale_rigid>
			<t_auto_padding></t_auto_padding>
			<auto_padding>on</auto_padding>
			<t_export></t_export>
			<export>on</export>
			<t_upper_limit></t_upper_limit>
			<upper_limit>100</upper_limit>
			<t_lower_limit></t_lower_limit>
			<lower_limit>0</lower_limit>
			<t_base_value></t_base_value>
			<base_value>1000</base_value>
			<t_unit_value></t_unit_value>
			<unit_value></unit_value>
			<t_unit_exponent_value></t_unit_exponent_value>
			<unit_exponent_value></unit_exponent_value>
			<t_vertical_label></t_vertical_label>
			<vertical_label>bytes</vertical_label>
		</graph>
		<items>
			<hash_100005556267797455ef3f8d4a1cdf6d20ae46>
				<task_item_id>hash_080005ee6d22b020f5a247862b508811716784</task_item_id>
				<color_id>FF0000</color_id>
				<graph_type_id>7</graph_type_id>
				<consolidation_function_id>1</consolidation_function_id>
				<cdef_id>hash_050005634a23af5e78af0964e8d33b1a4ed26b</cdef_id>
				<value></value>
				<gprint_id>hash_060005e9c43831e54eca8069317a2ce8c6f751</gprint_id>
				<text_format>Memory Used</text_format>
				<hard_return></hard_return>
				<sequence>1</sequence>
			</hash_100005556267797455ef3f8d4a1cdf6d20ae46>
			<hash_1000058e60e8408ca9685c031411d1c2842e20>
				<task_item_id>hash_080005ee6d22b020f5a247862b508811716784</task_item_id>
				<color_id>0</color_id>
				<graph_type_id>9</graph_type_id>
				<consolidation_function_id>4</consolidation_function_id>
				<cdef_id>hash_050005634a23af5e78af0964e8d33b1a4ed26b</cdef_id>
				<value></value>
				<gprint_id>hash_060005e9c43831e54eca8069317a2ce8c6f751</gprint_id>
				<text_format>Current:</text_format>
				<hard_return></hard_return>
				<sequence>2</sequence>
			</hash_1000058e60e8408ca9685c031411d1c2842e20>
			<hash_1000054dd4844b4c16bb6ce6327dc1d1cdaca8>
				<task_item_id>hash_080005ee6d22b020f5a247862b508811716784</task_item_id>
				<color_id>0</color_id>
				<graph_type_id>9</graph_type_id>
				<consolidation_function_id>1</consolidation_function_id>
				<cdef_id>hash_050005634a23af5e78af0964e8d33b1a4ed26b</cdef_id>
				<value></value>
				<gprint_id>hash_060005e9c43831e54eca8069317a2ce8c6f751</gprint_id>
				<text_format>Average:</text_format>
				<hard_return></hard_return>
				<sequence>3</sequence>
			</hash_1000054dd4844b4c16bb6ce6327dc1d1cdaca8>
			<hash_100005aea20d6c5e3354426e2d6d75627c77fe>
				<task_item_id>hash_080005ee6d22b020f5a247862b508811716784</task_item_id>
				<color_id>0</color_id>
				<graph_type_id>9</graph_type_id>
				<consolidation_function_id>3</consolidation_function_id>
				<cdef_id>hash_050005634a23af5e78af0964e8d33b1a4ed26b</cdef_id>
				<value></value>
				<gprint_id>hash_060005e9c43831e54eca8069317a2ce8c6f751</gprint_id>
				<text_format>Maximum:</text_format>
				<hard_return></hard_return>
				<sequence>4</sequence>
			</hash_100005aea20d6c5e3354426e2d6d75627c77fe>
		</items>
		<inputs>
			<hash_090005fac0d49f1bbe597cbd83bfc4a3209956>
				<name>Data Source [memory]</name>
				<description></description>
				<column_name>task_item_id</column_name>
				<items>hash_000005556267797455ef3f8d4a1cdf6d20ae46|hash_0000058e60e8408ca9685c031411d1c2842e20|hash_0000054dd4844b4c16bb6ce6327dc1d1cdaca8|hash_000005aea20d6c5e3354426e2d6d75627c77fe</items>
			</hash_090005fac0d49f1bbe597cbd83bfc4a3209956>
		</inputs>
	</hash_00000502d5e7579ab9b390756192bb43d3c1cd>
	<hash_0100053053e472f299be5161205d276755db34>
		<name>Mikrotik - SNMP - Memory Usage</name>
		<ds>
			<t_name></t_name>
			<name>|host_description| - Mikrotik - SNMP - Memory Usage</name>
			<data_input_id>hash_0300053eb92bb845b9660a7445cf9740726522</data_input_id>
			<t_rra_id></t_rra_id>
			<t_rrd_step></t_rrd_step>
			<rrd_step>300</rrd_step>
			<t_active></t_active>
			<active>on</active>
			<rra_items>hash_150005c21df5178e5c955013591239eb0afd46|hash_1500050d9c0af8b8acdc7807943937b3208e29|hash_1500056fc2d038fb42950138b0ce3e9874cc60|hash_150005e36f3adb9f152adfa5dc50fd2b23337e</rra_items>
		</ds>
		<items>
			<hash_080005ee6d22b020f5a247862b508811716784>
				<t_data_source_name></t_data_source_name>
				<data_source_name>memory</data_source_name>
				<t_rrd_minimum></t_rrd_minimum>
				<rrd_minimum>0</rrd_minimum>
				<t_rrd_maximum></t_rrd_maximum>
				<rrd_maximum>0</rrd_maximum>
				<t_data_source_type_id></t_data_source_type_id>
				<data_source_type_id>1</data_source_type_id>
				<t_rrd_heartbeat></t_rrd_heartbeat>
				<rrd_heartbeat>600</rrd_heartbeat>
				<t_data_input_field_id></t_data_input_field_id>
				<data_input_field_id>0</data_input_field_id>
			</hash_080005ee6d22b020f5a247862b508811716784>
		</items>
		<data>
			<item_000>
				<data_input_field_id>hash_07000592f5906c8dc0f964b41f4253df582c38</data_input_field_id>
				<t_value></t_value>
				<value></value>
			</item_000>
			<item_001>
				<data_input_field_id>hash_07000532285d5bf16e56c478f5e83f32cda9ef</data_input_field_id>
				<t_value></t_value>
				<value></value>
			</item_001>
			<item_002>
				<data_input_field_id>hash_070005ad14ac90641aed388139f6ba86a2e48b</data_input_field_id>
				<t_value></t_value>
				<value></value>
			</item_002>
			<item_003>
				<data_input_field_id>hash_0700059c55a74bd571b4f00a96fd4b793278c6</data_input_field_id>
				<t_value></t_value>
				<value></value>
			</item_003>
			<item_004>
				<data_input_field_id>hash_070005012ccb1d3687d3edb29c002ea66e72da</data_input_field_id>
				<t_value></t_value>
				<value></value>
			</item_004>
			<item_005>
				<data_input_field_id>hash_0700054276a5ec6e3fe33995129041b1909762</data_input_field_id>
				<t_value></t_value>
				<value>.1.3.6.1.2.1.25.2.3.1.6.2</value>
			</item_005>
		</data>
	</hash_0100053053e472f299be5161205d276755db34>
	<hash_0300053eb92bb845b9660a7445cf9740726522>
		<name>Get SNMP Data</name>
		<type_id>2</type_id>
		<input_string></input_string>
		<fields>
			<hash_07000592f5906c8dc0f964b41f4253df582c38>
				<name>SNMP IP Address</name>
				<update_rra></update_rra>
				<regexp_match></regexp_match>
				<allow_nulls></allow_nulls>
				<type_code>hostname</type_code>
				<input_output>in</input_output>
				<data_name>management_ip</data_name>
			</hash_07000592f5906c8dc0f964b41f4253df582c38>
			<hash_07000532285d5bf16e56c478f5e83f32cda9ef>
				<name>SNMP Community</name>
				<update_rra></update_rra>
				<regexp_match></regexp_match>
				<allow_nulls></allow_nulls>
				<type_code>snmp_community</type_code>
				<input_output>in</input_output>
				<data_name>snmp_community</data_name>
			</hash_07000532285d5bf16e56c478f5e83f32cda9ef>
			<hash_070005ad14ac90641aed388139f6ba86a2e48b>
				<name>SNMP Username</name>
				<update_rra></update_rra>
				<regexp_match></regexp_match>
				<allow_nulls>on</allow_nulls>
				<type_code>snmp_username</type_code>
				<input_output>in</input_output>
				<data_name>snmp_username</data_name>
			</hash_070005ad14ac90641aed388139f6ba86a2e48b>
			<hash_0700059c55a74bd571b4f00a96fd4b793278c6>
				<name>SNMP Password</name>
				<update_rra></update_rra>
				<regexp_match></regexp_match>
				<allow_nulls>on</allow_nulls>
				<type_code>snmp_password</type_code>
				<input_output>in</input_output>
				<data_name>snmp_password</data_name>
			</hash_0700059c55a74bd571b4f00a96fd4b793278c6>
			<hash_070005012ccb1d3687d3edb29c002ea66e72da>
				<name>SNMP Version (1, 2, or 3)</name>
				<update_rra></update_rra>
				<regexp_match></regexp_match>
				<allow_nulls>on</allow_nulls>
				<type_code>snmp_version</type_code>
				<input_output>in</input_output>
				<data_name>snmp_version</data_name>
			</hash_070005012ccb1d3687d3edb29c002ea66e72da>
			<hash_0700054276a5ec6e3fe33995129041b1909762>
				<name>OID</name>
				<update_rra></update_rra>
				<regexp_match></regexp_match>
				<allow_nulls></allow_nulls>
				<type_code>snmp_oid</type_code>
				<input_output>in</input_output>
				<data_name>oid</data_name>
			</hash_0700054276a5ec6e3fe33995129041b1909762>
		</fields>
	</hash_0300053eb92bb845b9660a7445cf9740726522>
	<hash_150005c21df5178e5c955013591239eb0afd46>
		<name>Daily (5 Minute Average)</name>
		<x_files_factor>0.5</x_files_factor>
		<steps>1</steps>
		<rows>600</rows>
		<timespan>86400</timespan>
		<cf_items>1|3</cf_items>
	</hash_150005c21df5178e5c955013591239eb0afd46>
	<hash_1500050d9c0af8b8acdc7807943937b3208e29>
		<name>Weekly (30 Minute Average)</name>
		<x_files_factor>0.5</x_files_factor>
		<steps>6</steps>
		<rows>700</rows>
		<timespan>604800</timespan>
		<cf_items>1|3</cf_items>
	</hash_1500050d9c0af8b8acdc7807943937b3208e29>
	<hash_1500056fc2d038fb42950138b0ce3e9874cc60>
		<name>Monthly (2 Hour Average)</name>
		<x_files_factor>0.5</x_files_factor>
		<steps>24</steps>
		<rows>775</rows>
		<timespan>2678400</timespan>
		<cf_items>1|3</cf_items>
	</hash_1500056fc2d038fb42950138b0ce3e9874cc60>
	<hash_150005e36f3adb9f152adfa5dc50fd2b23337e>
		<name>Yearly (1 Day Average)</name>
		<x_files_factor>0.5</x_files_factor>
		<steps>288</steps>
		<rows>797</rows>
		<timespan>33053184</timespan>
		<cf_items>1|3</cf_items>
	</hash_150005e36f3adb9f152adfa5dc50fd2b23337e>
	<hash_050005634a23af5e78af0964e8d33b1a4ed26b>
		<name>Multiply by 1024</name>
		<items>
			<hash_14000586370cfa0008fe8c56b28be80ee39a40>
				<sequence>1</sequence>
				<type>4</type>
				<value>CURRENT_DATA_SOURCE</value>
			</hash_14000586370cfa0008fe8c56b28be80ee39a40>
			<hash_1400059a35cc60d47691af37f6fddf02064e20>
				<sequence>2</sequence>
				<type>6</type>
				<value>1024</value>
			</hash_1400059a35cc60d47691af37f6fddf02064e20>
			<hash_1400055d7a7941ec0440b257e5598a27dd1688>
				<sequence>3</sequence>
				<type>2</type>
				<value>3</value>
			</hash_1400055d7a7941ec0440b257e5598a27dd1688>
		</items>
	</hash_050005634a23af5e78af0964e8d33b1a4ed26b>
	<hash_060005e9c43831e54eca8069317a2ce8c6f751>
		<name>Normal</name>
		<gprint_text>%8.2lf %s</gprint_text>
	</hash_060005e9c43831e54eca8069317a2ce8c6f751>
</cacti>
```

Kalo udah, coba balik ke tab Management > Devices, abis itu klik SNMP yang tadi udah dibuat.
![image](https://user-images.githubusercontent.com/100014814/161364485-80e641e6-75f8-4b3e-b9fb-7ad890516de5.png)

Scroll kebawah abis itu disini, tambahin Graph CPU Usage sama Memory Usage yang tadi udah kita tambahin.
![image](https://user-images.githubusercontent.com/100014814/161364972-f03e44ce-ce4c-4ffe-942c-68b1ef654241.png)

Nah kalo udah ditambahin, kalian tinggal perlu ke tab Graphs, abis itu klik dropdown di ujung kanan dan masukin "Place on a Tree (Default Tree)"
![image](https://user-images.githubusercontent.com/100014814/161365782-cd440cbc-a66c-41d0-a64a-98588cd77f52.png)

Setelah itu klik "Continue"
![image](https://user-images.githubusercontent.com/100014814/161365817-b3bc4c8c-fa24-403c-897f-93ebaa095f6e.png)

Terakhir, jika ingin melihat graph dari Mikrotik, klik "Graphs" pada kiri atas dan klik tree yang tadi kalian pilih (defaultnya di Default Tree).
![image](https://user-images.githubusercontent.com/100014814/161365884-ae257800-91fe-4029-b0cb-50bf17a068b5.png)

[NEXT](https://github.com/ujikomidn/Ujikom-IDN-2022/blob/main/Configuration/Lab8.md)
