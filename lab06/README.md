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

Проверка отношений смежности и таблиц маршрутизации:

--- R1 ----
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


--- R2 ----
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

--- R3 ----
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

