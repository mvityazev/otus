# Настройка EtherChannel

### Топология
![](eve.png)

### Таблица адресации
|Устройство|Интерфейс|IP-адрес     |Маска подсети|
|----------|---------|-------------|-------------|
|S1        |VLAN 99  |192.168.99.11|255.255.255.0|
|S2        |VLAN 99  |192.168.99.12|255.255.255.0|
|S3        |VLAN 99  |192.168.99.13|255.255.255.0|
|PC-A      |NIC      |192.168.10.1 |255.255.255.0|
|PC-B      |NIC      |192.168.10.2 |255.255.255.0|
|PC-C      |NIC      |192.168.10.3 |255.255.255.0|

### Задачи
Часть 1. Настройка базовых параметров коммутатора
Часть 2. Настройка PAgP
Часть 3. Настройка LACP

### Выполнение

#### 1. Настройка базовых параметров коммутатора
Выполняем базовую настройку на коммутаторах
```
conf t
hostname SX
no ip domain-lookup
enable secret class
line console 0
password cisco
login
exec-timeout 0 0
exit
line vty 0 4
password cisco
login
exit
line aux 0
password cisco
login
exit

service password-encryption
banner motd $ Authorized Access Only! $

vlan 99
name Managment
interface vlan 99
ip address 192.168.99.1X 255.255.255.0
no shutdown
exit

vlan 10
name Staff
exit

interface range ethernet 0/0-3  
shutdown
interface range ethernet 1/0-3  
shutdown

int ethernet 1/0
switchport mode access
switchport access vlan 10
no sh
```
