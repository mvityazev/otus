# Настройка HSRP

### Топология
![](eve.png)

### Таблица адресации
|Устройство|Интерфейс    |IP-адрес     |Маска подсети  |Шлюз по умолчанию|
|----------|-------------|-------------|---------------|-|
|R1        |E0/0         |192.168.1.1  |255.255.255.0  |-|
|          |S/1/0 (DCE)  |10.1.1.1     |255.255.255.252|-|
|R2        |S/1/0        |10.1.1.2     |255.255.255.252|-|
|          |S/1/1 (DCE)  |10.2.2.2     |255.255.255.252|-|
|          |Lo1          |209.165.200.225|255.255.255.224|-|
|R3        |E0/0         |192.168.1.3  |255.255.255.0  |-|
|          |S/1/1        |10.2.2.1     |255.255.255.252|-|
|S1        |VLAN 1       |192.168.1.11 |255.255.255.0  |192.168.1.1|
|S3        |VLAN 1       |192.168.1.13 |255.255.255.0  |192.168.1.3|
|PC-A      |NIC          |192.168.1.31 |255.255.255.0  |192.168.1.1|
|PC-C      |NIC          |192.168.1.33 |255.255.255.0  |192.168.1.3|

### Задачи
Часть 1. Построение сети и проверка соединения

Часть 2. Настройка обеспечения избыточности на первом хопе с помощью HSRP

### Выполнение

#### 1. Построение сети и проверка соединения
##### Выполняем базовую настройку маршрутизаторов и коммутаторов:
```
conf t
hostname RX (SX)
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

Настраиваем интерфейсы на маршрутизаторах:

-----R1 ------------
```
int serial1/0 
ip address 10.1.1.1 255.255.255.252
clock rate 128000
no sh
```
-----R2 ------------
```
int serial1/0
ip address 10.1.1.2 255.255.255.252
no sh
int serial1/1
ip address 10.2.2.2 255.255.255.252
clock rate 128000
no sh
```
-----R3 ------------
```
int serial1/1 
ip address 10.2.2.1 255.255.255.252
no sh
```


Настраиваем интерфейсы на коммутаторах:

--- S1 ----
```
vlan 1
interface vlan 1
ip address 192.168.1.11 255.255.255.0
no shutdown
exit
ip default-gateway 192.168.1.1
```

--- S3 ----
```
vlan 1
interface vlan 1
ip address 192.168.1.13 255.255.255.0
no shutdown
exit
ip default-gateway 192.168.1.3
```

##### Настройка RIP

--- R1 ----
```
router rip
version 2
network 192.168.1.0
network 10.0.0.0
```

--- R2 ----
```
router rip
version 2
network 10.0.0.0
default-information originate
```

--- R3 ----
```
router rip
version 2
network 192.168.1.0
network 10.0.0.0
```

Проверим таблицы маршрутизации:

--- R1 ----
```
R1(config)#do sh ip route
Codes: L - local, C - connected, S - static, R - RIP, M - mobile, B - BGP
       D - EIGRP, EX - EIGRP external, O - OSPF, IA - OSPF inter area 
       N1 - OSPF NSSA external type 1, N2 - OSPF NSSA external type 2
       E1 - OSPF external type 1, E2 - OSPF external type 2
       i - IS-IS, su - IS-IS summary, L1 - IS-IS level-1, L2 - IS-IS level-2
       ia - IS-IS inter area, * - candidate default, U - per-user static route
       o - ODR, P - periodic downloaded static route, H - NHRP, l - LISP
       a - application route
       + - replicated route, % - next hop override

Gateway of last resort is 10.1.1.2 to network 0.0.0.0

R*    0.0.0.0/0 [120/1] via 10.1.1.2, 00:00:21, Serial1/0
      10.0.0.0/8 is variably subnetted, 4 subnets, 3 masks
R        10.0.0.0/8 [120/1] via 192.168.1.3, 00:00:04, Ethernet0/0
C        10.1.1.0/30 is directly connected, Serial1/0
L        10.1.1.1/32 is directly connected, Serial1/0
R        10.2.2.0/30 [120/1] via 10.1.1.2, 00:00:21, Serial1/0
      192.168.1.0/24 is variably subnetted, 2 subnets, 2 masks
C        192.168.1.0/24 is directly connected, Ethernet0/0
L        192.168.1.1/32 is directly connected, Ethernet0/0
```

--- R2 ----
```
R2(config-router)#do sh ip route               
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

      10.0.0.0/8 is variably subnetted, 5 subnets, 3 masks
R        10.0.0.0/8 [120/2] via 10.2.2.1, 00:00:19, Serial1/1
                    [120/2] via 10.1.1.1, 00:00:20, Serial1/0
C        10.1.1.0/30 is directly connected, Serial1/0
L        10.1.1.2/32 is directly connected, Serial1/0
C        10.2.2.0/30 is directly connected, Serial1/1
L        10.2.2.2/32 is directly connected, Serial1/1
R     192.168.1.0/24 [120/1] via 10.2.2.1, 00:00:19, Serial1/1
                     [120/1] via 10.1.1.1, 00:00:20, Serial1/0
      209.165.200.0/24 is variably subnetted, 2 subnets, 2 masks
C        209.165.200.224/27 is directly connected, Loopback1
L        209.165.200.225/32 is directly connected, Loopback1
```

--- R3 ----
```
R3(config)#do sh ip route
Codes: L - local, C - connected, S - static, R - RIP, M - mobile, B - BGP
       D - EIGRP, EX - EIGRP external, O - OSPF, IA - OSPF inter area 
       N1 - OSPF NSSA external type 1, N2 - OSPF NSSA external type 2
       E1 - OSPF external type 1, E2 - OSPF external type 2
       i - IS-IS, su - IS-IS summary, L1 - IS-IS level-1, L2 - IS-IS level-2
       ia - IS-IS inter area, * - candidate default, U - per-user static route
       o - ODR, P - periodic downloaded static route, H - NHRP, l - LISP
       a - application route
       + - replicated route, % - next hop override

Gateway of last resort is 10.2.2.2 to network 0.0.0.0

R*    0.0.0.0/0 [120/1] via 10.2.2.2, 00:00:03, Serial1/1
      10.0.0.0/8 is variably subnetted, 4 subnets, 3 masks
R        10.0.0.0/8 [120/1] via 192.168.1.1, 00:00:18, Ethernet0/0
R        10.1.1.0/30 [120/1] via 10.2.2.2, 00:00:03, Serial1/1
C        10.2.2.0/30 is directly connected, Serial1/1
L        10.2.2.1/32 is directly connected, Serial1/1
      192.168.1.0/24 is variably subnetted, 2 subnets, 2 masks
C        192.168.1.0/24 is directly connected, Ethernet0/0
L        192.168.1.3/32 is directly connected, Ethernet0/0
```

### 2. Настройка обеспечения избыточности на первом хопе с помощью HSRP

Выполняем трассировку и ping запрос с PC-A:
```
PC-A> trace 209.165.200.225
trace to 209.165.200.225, 8 hops max, press Ctrl+C to stop
 1   192.168.1.1   0.228 ms  0.210 ms  0.216 ms
 2   *10.1.1.2   8.565 ms (ICMP type:3, code:3, Destination port unreachable)  *

PC-A> ping 209.165.200.225

84 bytes from 209.165.200.225 icmp_seq=1 ttl=254 time=8.651 ms
84 bytes from 209.165.200.225 icmp_seq=2 ttl=254 time=8.649 ms
84 bytes from 209.165.200.225 icmp_seq=3 ttl=254 time=8.668 ms
84 bytes from 209.165.200.225 icmp_seq=4 ttl=254 time=8.699 ms
84 bytes from 209.165.200.225 icmp_seq=5 ttl=254 time=8.708 ms

```

Выполняем трассировку и ping запрос с PC-C:
```
PC-C> trace 209.165.200.225
trace to 209.165.200.225, 8 hops max, press Ctrl+C to stop
 1   192.168.1.3   0.186 ms  0.161 ms  0.160 ms
 2   *10.2.2.2   8.516 ms (ICMP type:3, code:3, Destination port unreachable)  *

PC-C> ping 209.165.200.225

84 bytes from 209.165.200.225 icmp_seq=1 ttl=254 time=8.541 ms
84 bytes from 209.165.200.225 icmp_seq=2 ttl=254 time=8.453 ms
84 bytes from 209.165.200.225 icmp_seq=3 ttl=254 time=8.544 ms
84 bytes from 209.165.200.225 icmp_seq=4 ttl=254 time=8.546 ms
84 bytes from 209.165.200.225 icmp_seq=5 ttl=254 time=8.639 ms

```

При отключении интерфейса на соответствующем маршрутизаторе, пинг запрос перестает проходить.

#### Выполняем настройку HSRP

---R1----
```
interface e0/0
standby version 2
standby 1 ip 192.168.1.254
standby 1 priority 150
standby 1 preempt
```

---R3----
```
interface e0/0
standby version 2
standby 1 ip 192.168.1.254
```

Результат:

---R1----
```
R1(config)#do sh standby
Ethernet0/0 - Group 1 (version 2)
  State is Active
    2 state changes, last state change 00:02:43
  Virtual IP address is 192.168.1.254
  Active virtual MAC address is 0000.0c9f.f001
    Local virtual MAC address is 0000.0c9f.f001 (v2 default)
  Hello time 3 sec, hold time 10 sec
    Next hello sent in 0.624 secs
  Preemption enabled
  Active router is local
  Standby router is 192.168.1.3, priority 100 (expires in 8.688 sec)
  Priority 150 (configured 150)
  Group name is "hsrp-Et0/0-1" (default)
  
R1(config)#do sh standby brief
                     P indicates configured to preempt.
                     |
Interface   Grp  Pri P State   Active          Standby         Virtual IP
Et0/0       1    150 P Active  local           192.168.1.3     192.168.1.254
```

---R3----
```
R3(config)#do sh standby
Ethernet0/0 - Group 1 (version 2)
  State is Standby
    1 state change, last state change 00:01:23
  Virtual IP address is 192.168.1.254
  Active virtual MAC address is 0000.0c9f.f001
    Local virtual MAC address is 0000.0c9f.f001 (v2 default)
  Hello time 3 sec, hold time 10 sec
    Next hello sent in 0.560 secs
  Preemption disabled
  Active router is 192.168.1.1, priority 150 (expires in 9.312 sec)
    MAC address is aabb.cc00.2000
  Standby router is local
  Priority 100 (default 100)
  Group name is "hsrp-Et0/0-1" (default)
  
  R3(config)#do sh standby brief
                     P indicates configured to preempt.
                     |
Interface   Grp  Pri P State   Active          Standby         Virtual IP
Et0/0       1    100   Standby 192.168.1.1     local           192.168.1.254
```

#### Меняем шлюз по умолчанию на PC-A, PC-C, коммутаторах S1 и S3 на 192.168.1.254.

Проверяем сетевое взаимодействие:
```
PC-A> ping 192.168.1.33

84 bytes from 192.168.1.33 icmp_seq=1 ttl=64 time=0.217 ms
84 bytes from 192.168.1.33 icmp_seq=2 ttl=64 time=0.456 ms
84 bytes from 192.168.1.33 icmp_seq=3 ttl=64 time=0.367 ms
84 bytes from 192.168.1.33 icmp_seq=4 ttl=64 time=0.378 ms
84 bytes from 192.168.1.33 icmp_seq=5 ttl=64 time=0.415 ms

PC-A> ping 209.165.200.225

84 bytes from 209.165.200.225 icmp_seq=1 ttl=254 time=8.531 ms
84 bytes from 209.165.200.225 icmp_seq=2 ttl=254 time=8.773 ms
84 bytes from 209.165.200.225 icmp_seq=3 ttl=254 time=8.706 ms
84 bytes from 209.165.200.225 icmp_seq=4 ttl=254 time=8.870 ms
84 bytes from 209.165.200.225 icmp_seq=5 ttl=254 time=8.760 ms

```

#### Проверяем работу HSRP
```
VPCS> tracer 209.165.200.225
trace to 209.165.200.225, 8 hops max, press Ctrl+C to stop
 1   192.168.1.1   8.571 ms  8.472 ms  8.520 ms
 2   *10.1.1.2   8.430 ms (ICMP type:3, code:3, Destination port unreachable)  *
```

Отключаем интерфейс e/0/0 на R1:

```
VPCS> tracer 209.165.200.225 
trace to 209.165.200.225, 8 hops max, press Ctrl+C to stop
 1   192.168.1.3   0.470 ms  0.284 ms  0.300 ms
 2   *10.2.2.2   8.531 ms (ICMP type:3, code:3, Destination port unreachable)  *
```

При этом активным становится маршрутизатор R3:
```
R3(config)#do sh standby brief
                     P indicates configured to preempt.
                     |
Interface   Grp  Pri P State   Active          Standby         Virtual IP
Et0/0       1    100   Active  local           unknown         192.168.1.254
```

При изменении приоритета на R3, активным остается R1 до момента пока интерфейс Et0/0 доступен.
```
R3(config-if)#standby 1 priority 200
R3(config-if)#do sh standby brief   
                     P indicates configured to preempt.
                     |
Interface   Grp  Pri P State   Active          Standby         Virtual IP
Et0/0       1    200   Standby 192.168.1.1     local           192.168.1.254

```

```
R1(config-if)#sh

R1(config-if)#no sh
```

```
R3(config-if)#do sh standby brief
                     P indicates configured to preempt.
                     |
Interface   Grp  Pri P State   Active          Standby         Virtual IP
Et0/0       1    200   Active  local           192.168.1.1     192.168.1.254

```
