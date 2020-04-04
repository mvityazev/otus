# Настройка OSPFv2 для нескольких областей

### Топология
![](eve.png)

### Таблица адресации
|Устройство|Интерфейс    |IP-адрес       |Маска подсети   |
|----------|-------------|---------------|----------------|
|R1        |Lo0          |209.165.200.225|255.255.255.252 |
|          |Lo1          |192.168.1.1  |255.255.255.0     |
|          |Lo2          |192.168.2.1  |255.255.255.0     |
|          |S1/0 (DCE)   |192.168.12.1 |255.255.255.252   |
|R2        |Lo6          |192.168.6.1  |255.255.255.0     |
|          |S1/0         |192.168.12.2 |255.255.255.252   |
|          |S1/1 (DCE)   |192.168.23.1 |255.255.255.252   |
|R3        |Lo4          |192.168.4.1  |255.255.255.0     |
|          |Lo5          |192.168.5.1  |255.255.255.0     |
|          |S/1/1        |192.168.23.2 |255.255.255.252   |

### Задачи
1. Создание сети и настройка основных параметров устройства

2. Настройка сети OSPFv2 для нескольких областей

3. Настройка межобластных суммарных маршрутов

### Выполнение

#### 1. Создание сети и настройка основных параметров устройства

Выполняем базовую настройку маршрутизаторов:
```
conf t
hostname RX
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
```

Выполняем настройку интерфейсов:

-----R1 ------------
```
int lo0
ip address 209.165.200.225 255.255.255.252
no sh
int lo1
ip address 192.168.1.1 255.255.255.0
no sh
int lo2
ip address 192.168.2.1 255.255.255.0
no sh
int serial1/0 
ip address 192.168.12.1 255.255.255.252
clock rate 128000
bandwidth 128
no sh
```

-----R2 ------------
```
int lo6
ip address 192.168.6.1 255.255.255.0
no sh
int serial1/0 
ip address 192.168.12.2 255.255.255.252
no sh
int serial1/1
ip address 192.168.23.1 255.255.255.252
clock rate 128000
bandwidth 128
no sh
```

-----R3 ------------
```
int lo4
ip address 192.168.4.1 255.255.255.0
no sh
int lo5
ip address 192.168.5.1 255.255.255.0
no sh
int serial1/1 
ip address 192.168.23.2 255.255.255.252
no sh
```

#### 2. Настройка сети OSPFv2 для нескольких областей

Настройка OSPF:
--- R1 ----
```
ip route 0.0.0.0 0.0.0.0 lo0
router ospf 1
router-id 1.1.1.1
network 192.168.1.0 0.0.0.255 area 1
network 192.168.2.0 0.0.0.255 area 1
network 192.168.12.0 0.0.0.3 area 0
passive-interface Lo1 
passive-interface Lo2
default-information originate
```

--- R2 ----
```
router ospf 1
router-id 2.2.2.2
network 192.168.6.0 0.0.0.255 area 3
network 192.168.12.0 0.0.0.3 area 0
network 192.168.23.0 0.0.0.3 area 3
passive-interface Lo6
```

--- R3 ----
```
router ospf 1
router-id 3.3.3.3
network 192.168.4.0 0.0.0.255 area 3
network 192.168.5.0 0.0.0.255 area 3
network 192.168.23.0 0.0.0.3 area 3
passive-interface Lo4
passive-interface Lo5
```

Проверка корректности настройки OSPF:
```
R1(config)#do sh ip protocols 
*** IP Routing is NSF aware ***

Routing Protocol is "application"
  Sending updates every 0 seconds
  Invalid after 0 seconds, hold down 0, flushed after 0
  Outgoing update filter list for all interfaces is not set
  Incoming update filter list for all interfaces is not set
  Maximum path: 32
  Routing for Networks:
  Routing Information Sources:
    Gateway         Distance      Last Update
  Distance: (default is 4)

Routing Protocol is "ospf 1"
  Outgoing update filter list for all interfaces is not set
  Incoming update filter list for all interfaces is not set
  Router ID 1.1.1.1
  It is an area border and autonomous system boundary router
 Redistributing External Routes from,
  Number of areas in this router is 2. 2 normal 0 stub 0 nssa
  Maximum path: 4
  Routing for Networks:
    192.168.1.0 0.0.0.255 area 1
    192.168.2.0 0.0.0.255 area 1
    192.168.12.0 0.0.0.3 area 0
  Passive Interface(s):
    Loopback1
    Loopback2
  Routing Information Sources:
    Gateway         Distance      Last Update
    3.3.3.3              110      00:03:57
    2.2.2.2              110      00:07:09
  Distance: (default is 110)
```

```
R2(config)#do sh ip protocols 
*** IP Routing is NSF aware ***

Routing Protocol is "application"
  Sending updates every 0 seconds
  Invalid after 0 seconds, hold down 0, flushed after 0
  Outgoing update filter list for all interfaces is not set
  Incoming update filter list for all interfaces is not set
  Maximum path: 32
  Routing for Networks:
  Routing Information Sources:
    Gateway         Distance      Last Update
  Distance: (default is 4)

Routing Protocol is "ospf 1"
  Outgoing update filter list for all interfaces is not set
  Incoming update filter list for all interfaces is not set
  Router ID 2.2.2.2
  It is an area border router
  Number of areas in this router is 2. 2 normal 0 stub 0 nssa
  Maximum path: 4
  Routing for Networks:
    192.168.6.0 0.0.0.255 area 3
    192.168.12.0 0.0.0.3 area 0
    192.168.23.0 0.0.0.3 area 3
  Passive Interface(s):
    Loopback6
  Routing Information Sources:
    Gateway         Distance      Last Update
    3.3.3.3              110      00:00:16
    1.1.1.1              110      00:12:02
  Distance: (default is 110)
```

```
R3(config)#do sh ip protocols 
*** IP Routing is NSF aware ***

Routing Protocol is "application"
  Sending updates every 0 seconds
  Invalid after 0 seconds, hold down 0, flushed after 0
  Outgoing update filter list for all interfaces is not set
  Incoming update filter list for all interfaces is not set
  Maximum path: 32
  Routing for Networks:
  Routing Information Sources:
    Gateway         Distance      Last Update
  Distance: (default is 4)

Routing Protocol is "ospf 1"
  Outgoing update filter list for all interfaces is not set
  Incoming update filter list for all interfaces is not set
  Router ID 3.3.3.3
  Number of areas in this router is 1. 1 normal 0 stub 0 nssa
  Maximum path: 4
  Routing for Networks:
    192.168.4.0 0.0.0.255 area 3
    192.168.5.0 0.0.0.255 area 3
    192.168.23.0 0.0.0.3 area 3
  Passive Interface(s):
    Loopback4
    Loopback5
  Routing Information Sources:
    Gateway         Distance      Last Update
    1.1.1.1              110      00:01:05
    2.2.2.2              110      00:01:05
  Distance: (default is 110)
```

```
R1(config)#do sh ip ospf int brief 
Interface    PID   Area            IP Address/Mask    Cost  State Nbrs F/C
Se1/0        1     0               192.168.12.1/30    781   P2P   1/1
Lo1          1     1               192.168.1.1/24     1     LOOP  0/0
Lo2          1     1               192.168.2.1/24     1     LOOP  0/0
```

```
R2(config)#do sh ip ospf int brief 
Interface    PID   Area            IP Address/Mask    Cost  State Nbrs F/C
Se1/0        1     0               192.168.12.2/30    64    P2P   1/1
Lo6          1     3               192.168.6.1/24     1     LOOP  0/0
Se1/1        1     3               192.168.23.1/30    781   P2P   1/1
```

```
R3(config)#do sh ip ospf interface brief 
Interface    PID   Area            IP Address/Mask    Cost  State Nbrs F/C
Lo4          1     3               192.168.4.1/24     1     LOOP  0/0
Lo5          1     3               192.168.5.1/24     1     LOOP  0/0
Se1/1        1     3               192.168.23.2/30    64    P2P   1/1
```

##### Настройка аутентификации MD5
--- R1 ----
```
int serial1/0
ip ospf message-digest-key 1 md5 Cisco123
router ospf 1
area 0 authentication message-digest
```

--- R2 ----
```
int serial1/0
ip ospf message-digest-key 1 md5 Cisco123
int serial1/1
ip ospf message-digest-key 1 md5 Cisco123
router ospf 1
area 0 authentication message-digest
area 3 authentication message-digest
```

--- R3 ----
```
int serial1/1
ip ospf message-digest-key 1 md5 Cisco123
router ospf 1
area 3 authentication message-digest
```

Проверка:
```
R1(config)#do sh ip ospf neighbor 

Neighbor ID     Pri   State           Dead Time   Address         Interface
2.2.2.2           0   FULL/  -        00:00:37    192.168.12.2    Serial1/0
```

```
R2(config)#do sh ip ospf neighbor 

Neighbor ID     Pri   State           Dead Time   Address         Interface
1.1.1.1           0   FULL/  -        00:00:38    192.168.12.1    Serial1/0
3.3.3.3           0   FULL/  -        00:00:32    192.168.23.2    Serial1/1
```

```
R3(config)#do sh ip ospf neighbor 

Neighbor ID     Pri   State           Dead Time   Address         Interface
2.2.2.2           0   FULL/  -        00:00:37    192.168.23.1    Serial1/1
```

#### 3. Настройка межобластных суммарных маршрутов
