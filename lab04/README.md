# Настройка базового протокола OSPFv2 для одной области

### Топология
![](eve.png)

### Таблица адресации
|Устройство|Интерфейс    |IP-адрес     |Маска подсети  |Шлюз по умолчанию|
|----------|-------------|-------------|---------------|-|
|R1        |E0/0         |192.168.1.1  |255.255.255.0  |-|
|          |S/1/0 (DCE)  |192.168.12.1 |255.255.255.252|-|
|          |S/1/1        |192.168.13.1 |255.255.255.252|-|
|R2        |E0/0         |192.168.2.1  |255.255.255.0  |-|
|          |S/1/0        |192.168.12.2 |255.255.255.252|-|
|          |S/1/1 (DCE)  |192.168.23.1 |255.255.255.252|-|
|R3        |E0/0         |192.168.3.1  |255.255.255.0  |-|
|          |S/1/0 (DCE)  |192.168.13.2 |255.255.255.252|-|
|          |S/1/1        |192.168.23.2 |255.255.255.252|-|
|PC-A      |NIC          |192.168.1.3  |255.255.255.0  |192.168.1.1|
|PC-B      |NIC          |192.168.2.3  |255.255.255.0  |192.168.2.1|
|PC-C      |NIC          |192.168.3.3  |255.255.255.0  |192.168.3.1|

### Задачи
1. Создание сети и настройка основных параметров устройства

2. Настройка и проверка маршрутизации OSPF

3. Изменение назначений идентификаторов маршрутизаторов

4. Настройка пассивных интерфейсов OSPF

5. Изменение метрик OSPF

### Выполнение

#### 1. Создание сети и настройка основных параметров устройства

Базовая настройка маршрутизоаторов:
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

Настройка интерфейсов на R1:
```
int serial1/0 
ip address 192.168.12.1 255.255.255.252
clock rate 128000
no sh
int serial1/1 
ip address 192.168.13.1 255.255.255.252
no sh
interface e0/0
ip address 192.168.1.1 255.255.255.0
no shutdown
```


Настройка интерфейсов на R2:
```
int serial1/0 
ip address 192.168.12.2 255.255.255.252
no sh
int serial1/1 
ip address 192.168.23.1 255.255.255.252
clock rate 128000
no sh
interface e0/0
ip address 192.168.2.1 255.255.255.0
no shutdown
```


Настройка интерфейсов на R3:
```
int serial1/0 
ip address 192.168.13.2 255.255.255.252
clock rate 128000
no sh
int serial1/1 
ip address 192.168.23.2 255.255.255.252
no sh
interface e0/0
ip address 192.168.3.1 255.255.255.0
no shutdown
```

#### 2. Настройка и проверка маршрутизации OSPF
Настройка на R1:
```
router ospf 1
network 192.168.3.0 0.0.0.255 area 0
network 192.168.13.0 0.0.0.3 area 0
network 192.168.23.0 0.0.0.3 area 0
```

Настройка на R2:
```
router ospf 1
network 192.168.2.0 0.0.0.255 area 0
network 192.168.12.0 0.0.0.3 area 0
network 192.168.23.0 0.0.0.3 area 0
```

Настройка на R3:
```
router ospf 1
network 192.168.3.0 0.0.0.255 area 0
network 192.168.13.0 0.0.0.3 area 0
network 192.168.23.0 0.0.0.3 area 0
```

##### Провкрка работоспособности OSPF
```
R3(config-router)#do show ip ospf neighbor

Neighbor ID     Pri   State           Dead Time   Address         Interface
192.168.23.1      0   FULL/  -        00:00:34    192.168.23.1    Serial1/1
192.168.13.1      0   FULL/  -        00:00:31    192.168.13.1    Serial1/0
```

```
R3(config-router)#do show ip route
Codes: L - local, C - connected, S - static, R - RIP, M - mobile, B - BGP
       D - EIGRP, EX - EIGRP external, O - OSPF, IA - OSPF inter area 
       N1 - OSPF NSSA external type 1, N2 - OSPF NSSA external type 2
       E1 - OSPF external type 1, E2 - OSPF external type 2
       i - IS-IS, su - IS-IS summary, L1 - IS-IS level-1, L2 - IS-IS level-2
       ia - IS-IS inter area, * - candidate default, U - per-user static route
       o - ODR, P - periodic downloaded static route, H - NHRP, l - LISP
       a - application route
       + - replicated route, % - next hop override

Gateway of last resort is not set

O     192.168.1.0/24 [110/74] via 192.168.13.1, 00:08:23, Serial1/0
O     192.168.2.0/24 [110/74] via 192.168.23.1, 00:08:13, Serial1/1
      192.168.3.0/24 is variably subnetted, 2 subnets, 2 masks
C        192.168.3.0/24 is directly connected, Ethernet0/0
L        192.168.3.1/32 is directly connected, Ethernet0/0
      192.168.12.0/30 is subnetted, 1 subnets
O        192.168.12.0 [110/128] via 192.168.23.1, 00:08:13, Serial1/1
                      [110/128] via 192.168.13.1, 00:08:23, Serial1/0
      192.168.13.0/24 is variably subnetted, 2 subnets, 2 masks
C        192.168.13.0/30 is directly connected, Serial1/0
L        192.168.13.2/32 is directly connected, Serial1/0
      192.168.23.0/24 is variably subnetted, 2 subnets, 2 masks
C        192.168.23.0/30 is directly connected, Serial1/1
L        192.168.23.2/32 is directly connected, Serial1/1
```

```
R3(config-router)#do sh ip protocols
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
  Router ID 192.168.23.2
  Number of areas in this router is 1. 1 normal 0 stub 0 nssa
  Maximum path: 4
  Routing for Networks:
    192.168.3.0 0.0.0.255 area 0
    192.168.13.0 0.0.0.3 area 0
    192.168.23.0 0.0.0.3 area 0
  Routing Information Sources:
    Gateway         Distance      Last Update
    192.168.13.1         110      00:12:44
    192.168.23.1         110      00:12:34
  Distance: (default is 110)
```

```
R3(config)#do show ip ospf interface brief
Interface    PID   Area            IP Address/Mask    Cost  State Nbrs F/C
Se1/1        1     0               192.168.23.2/30    64    P2P   1/1
Se1/0        1     0               192.168.13.2/30    64    P2P   1/1
Et0/0        1     0               192.168.3.1/24     10    DR    0/0
```

Проверяем сетевое взаимодействие между PC:
```
PC-A> ping 192.168.2.3

84 bytes from 192.168.2.3 icmp_seq=1 ttl=62 time=9.030 ms
84 bytes from 192.168.2.3 icmp_seq=2 ttl=62 time=8.602 ms
84 bytes from 192.168.2.3 icmp_seq=3 ttl=62 time=8.581 ms
84 bytes from 192.168.2.3 icmp_seq=4 ttl=62 time=8.473 ms
84 bytes from 192.168.2.3 icmp_seq=5 ttl=62 time=8.691 ms

PC-A> ping 192.168.3.3

84 bytes from 192.168.3.3 icmp_seq=1 ttl=62 time=9.030 ms
84 bytes from 192.168.3.3 icmp_seq=2 ttl=62 time=8.430 ms
84 bytes from 192.168.3.3 icmp_seq=3 ttl=62 time=8.405 ms
84 bytes from 192.168.3.3 icmp_seq=4 ttl=62 time=8.467 ms
84 bytes from 192.168.3.3 icmp_seq=5 ttl=62 time=8.435 ms
```

#### 3. Изменение назначений идентификаторов маршрутизаторов
Настраиваем loopback адреса:


--- R1 ----
```
interface lo0
ip address 1.1.1.1 255.255.255.255
end
```

--- R2 ----
```
interface lo0
ip address 2.2.2.2 255.255.255.255
end
```

--- R3 ----
```
interface lo0
ip address 3.3.3.3 255.255.255.255
end
```

Проверка:
```
R1#sho ip protocols 
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
  Number of areas in this router is 1. 1 normal 0 stub 0 nssa
  Maximum path: 4
  Routing for Networks:
    192.168.1.0 0.0.0.255 area 0
    192.168.12.0 0.0.0.3 area 0
    192.168.13.0 0.0.0.3 area 0
  Routing Information Sources:
    Gateway         Distance      Last Update
    3.3.3.3              110      00:00:57
    2.2.2.2              110      00:00:57
  Distance: (default is 110)
```

```
R1#show ip ospf neighbor 

Neighbor ID     Pri   State           Dead Time   Address         Interface
3.3.3.3           0   FULL/  -        00:00:31    192.168.13.2    Serial1/1
2.2.2.2           0   FULL/  -        00:00:37    192.168.12.2    Serial1/0
```

Смена индентификатора через router-id:

--- R1 ----
```
router ospf 1
router-id 11.11.11.11
end
```

--- R2 ----
```
router ospf 1
router-id 22.22.22.22
end
```

--- R3 ----
```
router ospf 1
router-id 33.33.33.33
end
```

Проверка:
```
R1#show ip protocols 
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
  Router ID 11.11.11.11
  Number of areas in this router is 1. 1 normal 0 stub 0 nssa
  Maximum path: 4
  Routing for Networks:
    192.168.1.0 0.0.0.255 area 0
    192.168.12.0 0.0.0.3 area 0
    192.168.13.0 0.0.0.3 area 0
  Routing Information Sources:
    Gateway         Distance      Last Update
    33.33.33.33          110      00:00:17
    22.22.22.22          110      00:00:44
    3.3.3.3              110      00:02:53
    2.2.2.2              110      00:02:53
  Distance: (default is 110)
```

```
R1#show ip ospf neighbor 

Neighbor ID     Pri   State           Dead Time   Address         Interface
33.33.33.33       0   FULL/  -        00:00:30    192.168.13.2    Serial1/1
22.22.22.22       0   FULL/  -        00:00:31    192.168.12.2    Serial1/0
```

#### 4. Настройка пассивных интерфейсов OSPF
Выполняем настройку на R1:
```
router ospf 1
passive-interface e0/0
```

Проверяем, что eth0/0 стал пассивным:
```
R1(config-router)#do show ip ospf int e0/0
Ethernet0/0 is up, line protocol is up 
  Internet Address 192.168.1.1/24, Area 0, Attached via Network Statement
  Process ID 1, Router ID 11.11.11.11, Network Type BROADCAST, Cost: 10
  Topology-MTID    Cost    Disabled    Shutdown      Topology Name
        0           10        no          no            Base
  Transmit Delay is 1 sec, State DR, Priority 1
  Designated Router (ID) 11.11.11.11, Interface address 192.168.1.1
  No backup designated router on this network
  Timer intervals configured, Hello 10, Dead 40, Wait 40, Retransmit 5
    oob-resync timeout 40
    No Hellos (Passive interface) 
  Supports Link-local Signaling (LLS)
  Cisco NSF helper support enabled
  IETF NSF helper support enabled
  Index 1/1/1, flood queue length 0
  Next 0x0(0)/0x0(0)/0x0(0)
  Last flood scan length is 0, maximum is 0
  Last flood scan time is 0 msec, maximum is 0 msec
  Neighbor Count is 0, Adjacent neighbor count is 0 
  Suppress hello for 0 neighbor(s)
```

Настраиваем на R2 все интерфейсы, как пассивные:
```
router ospf 1
passive-interface default
```

В результате R2 пропадает на остальных маршрутизаторах:
```
R1(config-router)#do sh ip ospf neig

Neighbor ID     Pri   State           Dead Time   Address         Interface
33.33.33.33       0   FULL/  -        00:00:34    192.168.13.2    Serial1/1
```

Выполняем на R2:
```
router ospf 1
no passive-interface s1/0
no passive-interface s1/1
```

#### 5. Изменение метрик OSPF
```
R1(config)#do sh ip route ospf
Codes: L - local, C - connected, S - static, R - RIP, M - mobile, B - BGP
       D - EIGRP, EX - EIGRP external, O - OSPF, IA - OSPF inter area 
       N1 - OSPF NSSA external type 1, N2 - OSPF NSSA external type 2
       E1 - OSPF external type 1, E2 - OSPF external type 2
       i - IS-IS, su - IS-IS summary, L1 - IS-IS level-1, L2 - IS-IS level-2
       ia - IS-IS inter area, * - candidate default, U - per-user static route
       o - ODR, P - periodic downloaded static route, H - NHRP, l - LISP
       a - application route
       + - replicated route, % - next hop override

Gateway of last resort is not set

O     192.168.2.0/24 [110/74] via 192.168.12.2, 00:04:14, Serial1/0
O     192.168.3.0/24 [110/74] via 192.168.13.2, 00:17:57, Serial1/1
      192.168.23.0/30 is subnetted, 1 subnets
O        192.168.23.0 [110/128] via 192.168.13.2, 00:17:57, Serial1/1
                      [110/128] via 192.168.12.2, 00:04:14, Serial1/0
```

Суммарная стоимость маршрута равна 74 (e0/0 на R3 - 10 + s1/0 на R1 - 64).


Меняем параметр эталонной пропускной способности по умолчанию:
```
router ospf 1
auto-cost reference-bandwidth 10000
```

Проверяем:
```
R1(config)#do show ip route ospf
Codes: L - local, C - connected, S - static, R - RIP, M - mobile, B - BGP
       D - EIGRP, EX - EIGRP external, O - OSPF, IA - OSPF inter area 
       N1 - OSPF NSSA external type 1, N2 - OSPF NSSA external type 2
       E1 - OSPF external type 1, E2 - OSPF external type 2
       i - IS-IS, su - IS-IS summary, L1 - IS-IS level-1, L2 - IS-IS level-2
       ia - IS-IS inter area, * - candidate default, U - per-user static route
       o - ODR, P - periodic downloaded static route, H - NHRP, l - LISP
       a - application route
       + - replicated route, % - next hop override

Gateway of last resort is not set

O     192.168.2.0/24 [110/7476] via 192.168.12.2, 00:03:34, Serial1/0
O     192.168.3.0/24 [110/7476] via 192.168.13.2, 00:03:14, Serial1/1
      192.168.23.0/30 is subnetted, 1 subnets
O        192.168.23.0 [110/12952] via 192.168.13.2, 00:03:14, Serial1/1
                      [110/12952] via 192.168.12.2, 00:03:14, Serial1/0
```
Стоимость стала: 7476 (6476 + 1000).

Восстанавливаем прежнее значение:
```
router ospf 1
auto-cost reference-bandwidth 100
```

Изменим значение пропускной способность для интерфейса s1/0 и s1/1 на всех маршрутизаторах:
```
int s1/0
bandwidth 128
int s1/1
bandwidth 128
```

Результат:
```
R1(config)#do sh ip ospf interface brief 
Interface    PID   Area            IP Address/Mask    Cost  State Nbrs F/C
Et0/0        1     0               192.168.1.1/24     10    DR    0/0
Se1/1        1     0               192.168.13.1/30    781   P2P   1/1
Se1/0        1     0               192.168.12.1/30    781   P2P   1/1
```

```
R1(config)#do show ip route ospf
Codes: L - local, C - connected, S - static, R - RIP, M - mobile, B - BGP
       D - EIGRP, EX - EIGRP external, O - OSPF, IA - OSPF inter area 
       N1 - OSPF NSSA external type 1, N2 - OSPF NSSA external type 2
       E1 - OSPF external type 1, E2 - OSPF external type 2
       i - IS-IS, su - IS-IS summary, L1 - IS-IS level-1, L2 - IS-IS level-2
       ia - IS-IS inter area, * - candidate default, U - per-user static route
       o - ODR, P - periodic downloaded static route, H - NHRP, l - LISP
       a - application route
       + - replicated route, % - next hop override

Gateway of last resort is not set

O     192.168.2.0/24 [110/791] via 192.168.12.2, 00:01:00, Serial1/0
O     192.168.3.0/24 [110/791] via 192.168.13.2, 00:01:00, Serial1/1
      192.168.23.0/30 is subnetted, 1 subnets
O        192.168.23.0 [110/845] via 192.168.13.2, 00:01:00, Serial1/1
                      [110/845] via 192.168.12.2, 00:01:00, Serial1/0
```

Изменение стоимости маршрута на интерфейсе s1/1 на R1:
```
int s1/1
ip ospf cost 1565
```

Результат:
```
R1(config)#do sh ip route ospf
Codes: L - local, C - connected, S - static, R - RIP, M - mobile, B - BGP
       D - EIGRP, EX - EIGRP external, O - OSPF, IA - OSPF inter area 
       N1 - OSPF NSSA external type 1, N2 - OSPF NSSA external type 2
       E1 - OSPF external type 1, E2 - OSPF external type 2
       i - IS-IS, su - IS-IS summary, L1 - IS-IS level-1, L2 - IS-IS level-2
       ia - IS-IS inter area, * - candidate default, U - per-user static route
       o - ODR, P - periodic downloaded static route, H - NHRP, l - LISP
       a - application route
       + - replicated route, % - next hop override

Gateway of last resort is not set

O     192.168.2.0/24 [110/791] via 192.168.12.2, 00:00:21, Serial1/0
O     192.168.3.0/24 [110/1572] via 192.168.12.2, 00:00:04, Serial1/0
      192.168.23.0/30 is subnetted, 1 subnets
O        192.168.23.0 [110/1562] via 192.168.12.2, 00:00:21, Serial1/0
```
