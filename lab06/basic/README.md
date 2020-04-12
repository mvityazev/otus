# Базовая настройка протокола EIGRP для IPv4

### Топология
![](eve.png)

### Таблица адресации
|Устройство|Интерфейс    |IP-адрес     |Маска подсети  |Шлюз по умолчанию|
|----------|-------------|-------------|---------------|-|
|R1        |E0/0         |192.168.1.1  |255.255.255.0  |-|
|          |S/1/0 (DCE)  |10.1.1.1     |255.255.255.252|-|
|          |S/1/1        |10.3.3.1     |255.255.255.252|-|
|R2        |E0/0         |192.168.2.1  |255.255.255.0  |-|
|          |S/1/0        |10.1.1.2     |255.255.255.252|-|
|          |S/1/1 (DCE)  |10.2.2.2     |255.255.255.252|-|
|R3        |E0/0         |192.168.3.1  |255.255.255.0  |-|
|          |S/1/0 (DCE)  |10.3.3.2     |255.255.255.252|-|
|          |S/1/1        |10.2.2.1     |255.255.255.252|-|
|PC-A      |NIC          |192.168.1.3  |255.255.255.0  |192.168.1.1|
|PC-B      |NIC          |192.168.2.3  |255.255.255.0  |192.168.2.1|
|PC-C      |NIC          |192.168.3.3  |255.255.255.0  |192.168.3.1|

### Задачи
1. Построение сети и проверка соединения
2. Настройка маршрутизации EIGRP
3. Проверка маршрутизации EIGRP
4. Настройка пропускной способности и пассивных интерфейсов

### Выполнение

#### 1. Построение сети и проверка соединения
Выполним базовую настройку маршрутизаторов:
```
conf t
hostname X
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

Настройка интерфейсов:

--- R1 ----
```
int serial1/0 
ip address 10.1.1.1 255.255.255.252
clock rate 128000
no sh
int serial1/1 
ip address 10.3.3.1 255.255.255.252
no sh
interface e0/0
ip address 192.168.1.1 255.255.255.0
no shutdown
```

--- R2 ----
```
int serial1/0 
ip address 10.1.1.2 255.255.255.252
no sh
int serial1/1 
ip address 10.2.2.2 255.255.255.252
clock rate 128000
no sh
interface e0/0
ip address 192.168.2.1 255.255.255.0
no shutdown
```

--- R3 ----
```
int serial1/0 
ip address 10.3.3.2 255.255.255.252
clock rate 128000
no sh
int serial1/1 
ip address 10.2.2.1 255.255.255.252
no sh
interface e0/0
ip address 192.168.3.1 255.255.255.0
no shutdown
```

#### Почему рекомендуется использовать шаблонные маски при объявлении сетей? Можно ли исключить маску в какой-нибудь из вышеприведённых инструкций network? Если да, то в какой (в каких)?

Без указания шаблонной маски, EIGRP будет основываться на классовости IP-адреса. Исключить маску можно на IP-адресе 192.168.3.1, т.к. данная подсеть относится к C-классу.


#### 2. Настройка маршрутизации EIGRP
--- R1 ----
```
router eigrp 10
network 10.1.1.0 0.0.0.3
network 192.168.1.0 0.0.0.255
network 10.3.3.0 0.0.0.3
```

--- R2 ----
```
router eigrp 10
network 10.1.1.0 0.0.0.3
network 192.168.2.0 0.0.0.255
network 10.2.2.0 0.0.0.3
```

--- R3 ----
```
router eigrp 10
network 10.3.3.0 0.0.0.3
network 192.168.3.0 0.0.0.255
network 10.2.2.0 0.0.0.3
```


##### 3. Проверка маршрутизации EIGRP

```
R1(config)#do show ip eigrp neighbors 
EIGRP-IPv4 Neighbors for AS(10)
H   Address                 Interface              Hold Uptime   SRTT   RTO  Q  Seq
                                                   (sec)         (ms)       Cnt Num
1   10.3.3.2                Se1/1                    12 00:04:58   11   100  0  7
0   10.1.1.2                Se1/0                    11 00:05:07   12   100  0  9
```

```
R1(config)#do sh ip route eigrp 
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

      10.0.0.0/8 is variably subnetted, 5 subnets, 2 masks
D        10.2.2.0/30 [90/2681856] via 10.3.3.2, 00:08:06, Serial1/1
                     [90/2681856] via 10.1.1.2, 00:08:06, Serial1/0
D     192.168.2.0/24 [90/2195456] via 10.1.1.2, 00:08:06, Serial1/0
D     192.168.3.0/24 [90/2195456] via 10.3.3.2, 00:08:06, Serial1/1
```

```
R2(config)#do sh ip eigrp neighbors 
EIGRP-IPv4 Neighbors for AS(10)
H   Address                 Interface              Hold Uptime   SRTT   RTO  Q  Seq
                                                   (sec)         (ms)       Cnt Num
1   10.2.2.1                Se1/1                    13 00:06:11   15   100  0  8
0   10.1.1.1                Se1/0                    14 00:06:21   16   100  0  9
```

```
R2(config)#do sh ip route eigrp 
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

      10.0.0.0/8 is variably subnetted, 5 subnets, 2 masks
D        10.3.3.0/30 [90/2681856] via 10.2.2.1, 00:08:44, Serial1/1
                     [90/2681856] via 10.1.1.1, 00:08:44, Serial1/0
D     192.168.1.0/24 [90/2195456] via 10.1.1.1, 00:08:44, Serial1/0
D     192.168.3.0/24 [90/2195456] via 10.2.2.1, 00:08:44, Serial1/1
```

```
R3(config)#do show ip eigrp neighbors 
EIGRP-IPv4 Neighbors for AS(10)
H   Address                 Interface              Hold Uptime   SRTT   RTO  Q  Seq
                                                   (sec)         (ms)       Cnt Num
1   10.2.2.2                Se1/1                    13 00:06:43   23   138  0  8
0   10.3.3.1                Se1/0                    14 00:06:43   17   102  0  10
```

```
R3(config)#do sh ip route eigrp 
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

      10.0.0.0/8 is variably subnetted, 5 subnets, 2 masks
D        10.1.1.0/30 [90/2681856] via 10.3.3.1, 00:09:04, Serial1/0
                     [90/2681856] via 10.2.2.2, 00:09:04, Serial1/1
D     192.168.1.0/24 [90/2195456] via 10.3.3.1, 00:09:04, Serial1/0
D     192.168.2.0/24 [90/2195456] via 10.2.2.2, 00:09:04, Serial1/1
```

##### Таблица соседних устройств EIGRP
```
R1(config)#do sh ip eigrp topology 
EIGRP-IPv4 Topology Table for AS(10)/ID(192.168.1.1)
Codes: P - Passive, A - Active, U - Update, Q - Query, R - Reply,
       r - reply Status, s - sia Status 

P 192.168.3.0/24, 1 successors, FD is 2195456
        via 10.3.3.2 (2195456/281600), Serial1/1
P 192.168.2.0/24, 1 successors, FD is 2195456
        via 10.1.1.2 (2195456/281600), Serial1/0
P 10.2.2.0/30, 2 successors, FD is 2681856
        via 10.1.1.2 (2681856/2169856), Serial1/0
        via 10.3.3.2 (2681856/2169856), Serial1/1
P 10.3.3.0/30, 1 successors, FD is 2169856
        via Connected, Serial1/1
P 192.168.1.0/24, 1 successors, FD is 281600
        via Connected, Ethernet0/0
P 10.1.1.0/30, 1 successors, FD is 2169856
        via Connected, Serial1/0
```

#### Почему в таблице топологии маршрутизатора R1 отсутствуют возможные преемники?

Т.к. сети 192.168.1.0/24, 10.1.1.0/30 подключены непосредственно к R1 и другие беспетлевые маршруты отсутствуют.

```
R2(config)#do sh ip eigrp topology 
EIGRP-IPv4 Topology Table for AS(10)/ID(192.168.2.1)
Codes: P - Passive, A - Active, U - Update, Q - Query, R - Reply,
       r - reply Status, s - sia Status 

P 192.168.3.0/24, 1 successors, FD is 2195456
        via 10.2.2.1 (2195456/281600), Serial1/1
P 192.168.2.0/24, 1 successors, FD is 281600
        via Connected, Ethernet0/0
P 10.2.2.0/30, 1 successors, FD is 2169856
        via Connected, Serial1/1
P 10.3.3.0/30, 2 successors, FD is 2681856
        via 10.1.1.1 (2681856/2169856), Serial1/0
        via 10.2.2.1 (2681856/2169856), Serial1/1
P 192.168.1.0/24, 1 successors, FD is 2195456
        via 10.1.1.1 (2195456/281600), Serial1/0
P 10.1.1.0/30, 1 successors, FD is 2169856
        via Connected, Serial1/0
```

```
R3(config)#do sh ip eigrp topology 
EIGRP-IPv4 Topology Table for AS(10)/ID(192.168.3.1)
Codes: P - Passive, A - Active, U - Update, Q - Query, R - Reply,
       r - reply Status, s - sia Status 

P 192.168.3.0/24, 1 successors, FD is 281600
        via Connected, Ethernet0/0
P 192.168.2.0/24, 1 successors, FD is 2195456
        via 10.2.2.2 (2195456/281600), Serial1/1
P 10.2.2.0/30, 1 successors, FD is 2169856
        via Connected, Serial1/1
P 10.3.3.0/30, 1 successors, FD is 2169856
        via Connected, Serial1/0
P 192.168.1.0/24, 1 successors, FD is 2195456
        via 10.3.3.1 (2195456/281600), Serial1/0
P 10.1.1.0/30, 2 successors, FD is 2681856
        via 10.2.2.2 (2681856/2169856), Serial1/1
        via 10.3.3.1 (2681856/2169856), Serial1/0
```

##### Параметры маршрутизации EIGRP и объявленные сети
```
R1(config)#do sh ip protocol
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

Routing Protocol is "eigrp 10"
  Outgoing update filter list for all interfaces is not set
  Incoming update filter list for all interfaces is not set
  Default networks flagged in outgoing updates
  Default networks accepted from incoming updates
  EIGRP-IPv4 Protocol for AS(10)
    Metric weight K1=1, K2=0, K3=1, K4=0, K5=0
    Soft SIA disabled
    NSF-aware route hold timer is 240
    Router-ID: 192.168.1.1
    Topology : 0 (base) 
      Active Timer: 3 min
      Distance: internal 90 external 170
      Maximum path: 4
      Maximum hopcount 100
      Maximum metric variance 1

  Automatic Summarization: disabled
  Maximum path: 4
  Routing for Networks:
    10.1.1.0/30
    10.3.3.0/30
    192.168.1.0
  Routing Information Sources:
    Gateway         Distance      Last Update
    10.3.3.2              90      01:22:31
    10.1.1.2              90      01:22:31
  Distance: internal 90 external 170
```


Номер автономной сети: 10

Анонсируются сети: 10.1.1.0/30, 10.3.3.0/30, 192.168.1.0/24

Значение административной дистанции: 90

Кол-во маршрутов с равной стоймостью: 4


#### 4. Настройка пропускной способности и пассивных интерфейсов

Выполняем настройку на маршрутизаторах:

--- R1 ----
```
interface s1/0
bandwidth 2000
interface s1/1
bandwidth 64
router eigrp 10
passive-interface e0/0
```

--- R2 ----
```
interface s1/0
bandwidth 2000
interface s1/1
bandwidth 2000
router eigrp 10
passive-interface e0/0
```

--- R3 ----
```
interface s1/0
bandwidth 64
interface s1/1
bandwidth 2000
router eigrp 10
passive-interface e0/0
```

Проверка:
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

Gateway of last resort is not set

      10.0.0.0/8 is variably subnetted, 5 subnets, 2 masks
C        10.1.1.0/30 is directly connected, Serial1/0
L        10.1.1.1/32 is directly connected, Serial1/0
D        10.2.2.0/30 [90/2304000] via 10.1.1.2, 00:02:46, Serial1/0
C        10.3.3.0/30 is directly connected, Serial1/1
L        10.3.3.1/32 is directly connected, Serial1/1
      192.168.1.0/24 is variably subnetted, 2 subnets, 2 masks
C        192.168.1.0/24 is directly connected, Ethernet0/0
L        192.168.1.1/32 is directly connected, Ethernet0/0
D     192.168.2.0/24 [90/1817600] via 10.1.1.2, 00:02:52, Serial1/0
D     192.168.3.0/24 [90/2329600] via 10.1.1.2, 00:02:46, Serial1/0
```

```
R2(config)#do sh ip route
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

      10.0.0.0/8 is variably subnetted, 5 subnets, 2 masks
C        10.1.1.0/30 is directly connected, Serial1/0
L        10.1.1.2/32 is directly connected, Serial1/0
C        10.2.2.0/30 is directly connected, Serial1/1
L        10.2.2.2/32 is directly connected, Serial1/1
D        10.3.3.0/30 [90/41024000] via 10.2.2.1, 00:14:14, Serial1/1
                     [90/41024000] via 10.1.1.1, 00:14:14, Serial1/0
D     192.168.1.0/24 [90/1817600] via 10.1.1.1, 00:14:14, Serial1/0
      192.168.2.0/24 is variably subnetted, 2 subnets, 2 masks
C        192.168.2.0/24 is directly connected, Ethernet0/0
L        192.168.2.1/32 is directly connected, Ethernet0/0
D     192.168.3.0/24 [90/1817600] via 10.2.2.1, 00:14:26, Serial1/1
```

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

Gateway of last resort is not set

      10.0.0.0/8 is variably subnetted, 5 subnets, 2 masks
D        10.1.1.0/30 [90/2304000] via 10.2.2.2, 00:14:35, Serial1/1
C        10.2.2.0/30 is directly connected, Serial1/1
L        10.2.2.1/32 is directly connected, Serial1/1
C        10.3.3.0/30 is directly connected, Serial1/0
L        10.3.3.2/32 is directly connected, Serial1/0
D     192.168.1.0/24 [90/2329600] via 10.2.2.2, 00:14:35, Serial1/1
D     192.168.2.0/24 [90/1817600] via 10.2.2.2, 00:14:35, Serial1/1
      192.168.3.0/24 is variably subnetted, 2 subnets, 2 masks
C        192.168.3.0/24 is directly connected, Ethernet0/0
L        192.168.3.1/32 is directly connected, Ethernet0/0
```

#### При выполнении лабораторной работы можно было ограничиться только статической маршрутизацией. Каковы преимущества использования EIGRP?
Преимущества EIGRP, как и остальных протоколов динамической маршрутизации, в отсутствии необходимости ручной поддержки в актуальном состоянии таблиц маршрутизации. Также, с помощью EIGRP, можно реализовать автоматическое резервирование каналов связи.
