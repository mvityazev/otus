# Поиск и устранение неполадок в работе EtherChannel

### Топология
![](eve.png)

### Таблица адресации
|Устройство|Интерфейс|IP-адрес     |Маска подсети|
|----------|---------|-------------|-------------|
|S1        |VLAN 99  |192.168.1.11 |255.255.255.0|
|S2        |VLAN 99  |192.168.1.12 |255.255.255.0|
|S3        |VLAN 99  |192.168.1.13 |255.255.255.0|
|PC-A      |NIC      |192.168.0.1  |255.255.255.0|
|PC-C      |NIC      |192.168.0.3  |255.255.255.0|

### Задачи
Часть 1. Построение сети и загрузка настроек устройств

Часть 2. Отладка EtherChannel

### Выполнение

#### 1. Построение сети и загрузка настроек устройств
Первоначальная конфигурация коммутаторов:

---- S1 ----
```
hostname S1
interface range e0/0-3, e1/0-3
shutdown
exit
enable secret class
no ip domain lookup
line vty 0 4
password cisco
login
line con 0
 password cisco
 logging synchronous
 login
 exit
vlan 10
 name User
vlan 99
 Name Management

interface range e0/0-1
switchport trunk encapsulation dot1q
 switchport mode trunk
 channel-group 1 mode active
 switchport trunk native vlan 99
 no shutdown

interface range e0/2-3
 channel-group 2 mode desirable
 switchport trunk encapsulation dot1q
 switchport trunk native vlan 99
 no shutdown

interface e1/0
 switchport mode access
 switchport access vlan 10
 no shutdown

interface vlan 99
 ip address 192.168.1.11 255.255.255.0

interface port-channel 1
 switchport trunk native vlan 99
 switchport trunk encapsulation dot1q
 switchport mode trunk
interface port-channel 2
 switchport trunk native vlan 99
 switchport mode access
```

---- S2 ----
```
hostname S2
interface range e0/0-3, e1/0-3
 shutdown
 exit
enable secret class
no ip domain lookup

line vty 0 4
 password cisco
 login
line con 0
 password cisco
 logging synchronous
 login
 exit

vlan 10
 name User
vlan 99
 name Management

spanning-tree vlan 1,10,99 root primary

interface range e0/0-1
 switchport trunk encapsulation dot1q
 switchport mode trunk
 channel-group 1 mode desirable
 switchport trunk native vlan 99
 no shutdown

interface range e0/2-3
 switchport trunk encapsulation dot1q
 switchport mode trunk
 channel-group 3 mode desirable
 switchport trunk native vlan 99

interface vlan 99
 ip address 192.168.1.12 255.255.255.0

interface port-channel 1
 switchport trunk native vlan 99
 switchport trunk allowed vlan 1,99

interface port-channel 3
 switchport trunk native vlan 99
 switchport trunk allowed vlan 1,10,99
 switchport trunk encapsulation dot1q
 switchport mode trunk
```

---- S3 ----
```
hostname S3
interface range e0/0-3, e1/0-3
 shutdown
 exit
enable secret class
no ip domain lookup

line vty 0 4
 password cisco
 login
line con 0
 password cisco
 logging synchronous
 login
 exit

vlan 10
 name User

vlan 99
 name Management

interface range e0/0-1

interface range e0/2-3
 switchport trunk encapsulation dot1q
 switchport mode trunk
 channel-group 3 mode desirable
 switchport trunk native vlan 99
 no shutdown

interface e1/0
 switchport mode access
 switchport access vlan 10
 no shutdown

interface vlan 99
 ip address 192.168.1.13 255.255.255.0

interface port-channel 3
 switchport trunk encapsulation dot1q
 switchport trunk native vlan 99
 switchport mode trunk
```

### 2. Поиск и устранение неисправностей в EtherChannel
Выполняем show interfaces trunk на коммутаторах:
```
S1(config-if)#do sh int trunk

Port        Mode             Encapsulation  Status        Native vlan
Et0/0       on               802.1q         trunking      99
Et0/1       on               802.1q         trunking      99

Port        Vlans allowed on trunk
Et0/0       none
Et0/1       none

Port        Vlans allowed and active in management domain
Et0/0       none
Et0/1       none

Port        Vlans in spanning tree forwarding state and not pruned
Et0/0       none
Et0/1       none

```

```
S2(config-if)#do sh int tru

Port        Mode             Encapsulation  Status        Native vlan
Et0/0       on               802.1q         trunking      99
Et0/1       on               802.1q         trunking      99

Port        Vlans allowed on trunk
Et0/0       1,99
Et0/1       1,99

Port        Vlans allowed and active in management domain
Et0/0       1,99
Et0/1       1,99

Port        Vlans in spanning tree forwarding state and not pruned
Et0/0       none
Et0/1       none
```

```
S3(config-if)#do sh int trun

Port        Mode             Encapsulation  Status        Native vlan
Po3         on               802.1q         trunking      99

Port        Vlans allowed on trunk
Po3         1-4094

Port        Vlans allowed and active in management domain
Po3         1,10,99

Port        Vlans in spanning tree forwarding state and not pruned
Po3         none
```
