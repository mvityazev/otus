# Настройка расширенных функций EIGRP для IPv4

### Топология
![](eve.png)

### Таблица адресации
|Устройство|Интерфейс    |IP-адрес     |Маска подсети  |Шлюз по умолчанию|
|----------|-------------|-------------|---------------|-  |
|R1        |E0/0         |192.168.1.1  |255.255.255.0  |-  |
|          |S/1/0 (DCE)  |192.168.12.1 |255.255.255.252|-  |
|          |S/1/1        |192.168.13.1 |255.255.255.252|-  |
|          |Lo1          |192.168.11.1 |255.255.255.252|Н/Д|
|          |Lo5          |192.168.11.5 |255.255.255.252|-  |
|          |Lo9          |192.168.11.9 |255.255.255.252|-  |
|          |Lo13         |192.168.11.13|255.255.255.252|-  |
|R2        |E0/0         |192.168.2.1  |255.255.255.0  |-  |
|          |S/1/0        |192.168.12.2 |255.255.255.252|-  |
|          |S/1/1 (DCE)  |192.168.23.1 |255.255.255.252|-  |
|          |Lo1          |192.168.22.1 |255.255.255.252|-  |
|R3        |E0/0         |192.168.3.1  |255.255.255.0  |-  |
|          |S/1/0 (DCE)  |192.168.13.2 |255.255.255.252|-  |
|          |S/1/1        |192.168.23.2 |255.255.255.252|-  |
|          |Lo1          |192.168.33.1 |255.255.255.252|Н/Д|
|          |Lo5          |192.168.33.5 |255.255.255.252|-  |
|          |Lo9          |192.168.33.9 |255.255.255.252|-  |
|          |Lo13         |192.168.33.13|255.255.255.252|-  |
|PC-A      |NIC          |192.168.1.3  |255.255.255.0  |192.168.1.1|
|PC-B      |NIC          |192.168.2.3  |255.255.255.0  |192.168.2.1|
|PC-C      |NIC          |192.168.3.3  |255.255.255.0  |192.168.3.1|

### Задачи

Часть 1. Создание сети и настройка основных параметров устройства
Часть 2. Настройка EIGRP и проверка подключения
Часть 3. Настройка EIGRP для автоматического объединения
Часть 4. Настройка и распространение статического маршрута по умолчанию
Часть 5. Выполнение точной настройки EIGRP

### Выполнение

#### 1. Построение сети и проверка соединения
Выполним базовую настройку маршрутизаторов:
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

Выполним настройку интерфейсов.

-----R1 ------------
```
int serial1/0 
ip address 192.168.12.1 255.255.255.252
clock rate 1024000
no shutdown
int serial1/1 
ip address 192.168.13.1 255.255.255.252
no shutdown
interface e0/0
ip address 192.168.1.1 255.255.255.0
no shutdown
```

-----R2 ------------
```
int serial1/0 
ip address 19.168.12.2 255.255.255.252
no shutdown
int serial1/1 
ip address 192.168.23.1 255.255.255.252
clock rate 64000
no shutdown
interface e0/0
ip address 192.168.2.1 255.255.255.0
no shutdown
```

-----R3 ------------
```
int serial1/0 
ip address 192.168.13.2 255.255.255.252
clock rate 1544000
no shutdown
int serial1/1 
ip address 192.168.23.2 255.255.255.252
no shutdown
interface e0/0
ip address 192.168.3.1 255.255.255.0
no shutdown
```

#### 2. Настройка EIGRP и проверка подключения

Настройка EIGRP для всех сетей с прямым подключением:

--- R1 ----
```
router eigrp 1
network 192.168.1.0 0.0.0.255
network 192.168.12.0 0.0.0.3
network 192.168.13.0 0.0.0.3
passive-interface e0/0

interface s1/0
bandwidth 1024
interface s1/1
bandwidth 64
```

--- R2 ----
```
router eigrp 1
network 192.168.2.0 0.0.0.255
network 192.168.12.0 0.0.0.3
network 192.168.23.0 0.0.0.3
passive-interface e0/0

interface s1/0
bandwidth 1024
```

--- R3 ----
```
router eigrp 1
network 192.168.3.0 0.0.0.255
network 192.168.13.0 0.0.0.3
network 192.168.23.0 0.0.0.3
passive-interface e0/0

interface s1/0
bandwidth 64
```

Проверим сетевое взаимодействие:
```
PC-C> ping 192.168.1.3

84 bytes from 192.168.1.3 icmp_seq=1 ttl=62 time=24.842 ms
84 bytes from 192.168.1.3 icmp_seq=2 ttl=62 time=24.802 ms
84 bytes from 192.168.1.3 icmp_seq=3 ttl=62 time=24.195 ms
84 bytes from 192.168.1.3 icmp_seq=4 ttl=62 time=24.773 ms
84 bytes from 192.168.1.3 icmp_seq=5 ttl=62 time=24.783 ms

PC-C> ping 192.168.2.3

84 bytes from 192.168.2.3 icmp_seq=1 ttl=62 time=8.731 ms
84 bytes from 192.168.2.3 icmp_seq=2 ttl=62 time=8.489 ms
84 bytes from 192.168.2.3 icmp_seq=3 ttl=62 time=8.499 ms
84 bytes from 192.168.2.3 icmp_seq=4 ttl=62 time=8.525 ms
84 bytes from 192.168.2.3 icmp_seq=5 ttl=62 time=8.477 ms
```

#### 3. Настройка EIGRP для автоматического объединения

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

Routing Protocol is "eigrp 1"
  Outgoing update filter list for all interfaces is not set
  Incoming update filter list for all interfaces is not set
  Default networks flagged in outgoing updates
  Default networks accepted from incoming updates
  EIGRP-IPv4 Protocol for AS(1)
    Metric weight K1=1, K2=0, K3=1, K4=0, K5=0
    Soft SIA disabled
    NSF-aware route hold timer is 240
    Router-ID: 192.168.13.1
    Topology : 0 (base) 
      Active Timer: 3 min
      Distance: internal 90 external 170
      Maximum path: 4
      Maximum hopcount 100
      Maximum metric variance 1

  Automatic Summarization: disabled
  Maximum path: 4
  Routing for Networks:
    192.168.1.0
    192.168.12.0/30
    192.168.13.0/30
  Passive Interface(s):
    Ethernet0/0
  Routing Information Sources:
    Gateway         Distance      Last Update
    192.168.13.2          90      00:07:46
  Distance: internal 90 external 170
```

По-умолчанию автоматическое объединение выключено.

Настроим loopback интерфейсы на R1:
```
interface lo1
ip address 192.168.11.1 255.255.255.252
end
interface lo5
ip address 192.168.11.5 255.255.255.252
end
interface lo9
ip address 192.168.11.9 255.255.255.252
end
interface lo13
ip address 192.168.11.13 255.255.255.252
end

router eigrp 1
network 192.168.11.0 0.0.0.3
network 192.168.11.4 0.0.0.3
network 192.168.11.8 0.0.0.3
network 192.168.11.12 0.0.0.3
```

Проверим, как отображаются сети loopback на R2:
```
R2(config)#do show ip route eigrp
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

D     192.168.1.0/24 [90/41049600] via 192.168.23.2, 00:22:48, Serial1/1
D     192.168.3.0/24 [90/2195456] via 192.168.23.2, 00:31:41, Serial1/1
      192.168.11.0/30 is subnetted, 4 subnets
D        192.168.11.0 [90/41152000] via 192.168.23.2, 00:01:37, Serial1/1
D        192.168.11.4 [90/41152000] via 192.168.23.2, 00:00:33, Serial1/1
D        192.168.11.8 [90/41152000] via 192.168.23.2, 00:00:08, Serial1/1
D        192.168.11.12 [90/41152000] via 192.168.23.2, 00:00:06, Serial1/1
      192.168.12.0/30 is subnetted, 1 subnets
D        192.168.12.0 [90/41536000] via 192.168.23.2, 00:22:48, Serial1/1
      192.168.13.0/30 is subnetted, 1 subnets
D        192.168.13.0 [90/41024000] via 192.168.23.2, 00:22:48, Serial1/1
```

Каждая подсеть 192.168.11/30 отображается отдельно, включим суммаризацию на R1:
```
router eigrp 1
auto-summary
```

Повторно проверим таблицу маршрутизацию на R2:
```
R2(config)#do show ip route eigrp
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

D     192.168.1.0/24 [90/41049600] via 192.168.23.2, 00:27:48, Serial1/1
D     192.168.3.0/24 [90/2195456] via 192.168.23.2, 00:36:41, Serial1/1
      192.168.11.0/24 is subnetted, 1 subnets
D        192.168.11.0 [90/41152000] via 192.168.23.2, 00:00:48, Serial1/1
      192.168.12.0/24 is subnetted, 1 subnets
D        192.168.12.0 [90/41536000] via 192.168.23.2, 00:00:48, Serial1/1
      192.168.13.0/30 is subnetted, 1 subnets
D        192.168.13.0 [90/41024000] via 192.168.23.2, 00:27:48, Serial1/1
```
Подсети 192.168.111/30 объединены до /24.

Настроим loopback интерфейсы на R3:
```
interface lo1
ip address 192.168.33.1 255.255.255.252
interface lo5
ip address 192.168.33.5 255.255.255.252
interface lo9
ip address 192.168.33.9 255.255.255.252
interface lo13
ip address 192.168.33.13 255.255.255.252

router eigrp 1
network 192.168.33.0 0.0.0.3
network 192.168.33.4 0.0.0.3
network 192.168.33.8 0.0.0.3
network 192.168.33.12 0.0.0.3
```

Проверим таблицу маршрутизации на R2:
```
R2(config)#do show ip route eigrp
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

D     192.168.1.0/24 [90/41049600] via 192.168.23.2, 00:36:48, Serial1/1
D     192.168.3.0/24 [90/2195456] via 192.168.23.2, 00:45:41, Serial1/1
      192.168.11.0/24 is subnetted, 1 subnets
D        192.168.11.0 [90/41152000] via 192.168.23.2, 00:09:48, Serial1/1
      192.168.12.0/24 is subnetted, 1 subnets
D        192.168.12.0 [90/41536000] via 192.168.23.2, 00:09:48, Serial1/1
      192.168.13.0/30 is subnetted, 1 subnets
D        192.168.13.0 [90/41024000] via 192.168.23.2, 00:36:48, Serial1/1
      192.168.33.0/30 is subnetted, 4 subnets
D        192.168.33.0 [90/2297856] via 192.168.23.2, 00:00:04, Serial1/1
```

#### 4. Настройка и распространение статического маршрута по умолчанию

Настройка loopback интерфейса на R2:
```
interface lo1
ip address 192.168.22.1 255.255.255.252
ip route 0.0.0.0 0.0.0.0 Lo1
router eigrp 1
redistribute static
```

Проверка:
```
R1(config)#do sh ip route eigrp | include 0.0.0.0
Gateway of last resort is 192.168.13.2 to network 0.0.0.0
D*EX  0.0.0.0/0 [170/41152000] via 192.168.13.2, 00:03:12, Serial1/1
```

Административная дистанция - 170.

#### 5. Выполнение точной настройки EIGRP

##### Настройка параметров использования пропускной способности.

--- R1 ----
```
interface s1/0
ip bandwidth-percent eigrp 1 75
interface s1/1
ip bandwidth-percent eigrp 1 40
```

--- R2 ----
```
interface s1/0
ip bandwidth-percent eigrp 1 75
```

--- R3 ----
```
interface s1/0
ip bandwidth-percent eigrp 1 40
```

##### Настройка интервалов отправки пакетов приветствия и таймер удержания.

Текущие настройки:
```
R2(config)#do show ip eigrp interfaces detail
EIGRP-IPv4 Interfaces for AS(1)
                              Xmit Queue   PeerQ        Mean   Pacing Time   Multicast    Pending
Interface              Peers  Un/Reliable  Un/Reliable  SRTT   Un/Reliable   Flow Timer   Routes
Se1/1                    1        0/0       0/0          11       0/16          60           0
  Hello-interval is 5, Hold-time is 15
  Split-horizon is enabled
  Next xmit serial <none>
  Packetized sent/expedited: 13/0
  Hello's sent/expedited: 862/2
  Un/reliable mcasts: 0/0  Un/reliable ucasts: 15/14
  Mcast exceptions: 0  CR packets: 0  ACKs suppressed: 0
  Retransmissions sent: 0  Out-of-sequence rcvd: 0
  Topology-ids on interface - 0 
  Authentication mode is not set
```
Таймер приветствия по умолчанию: 5.

Таймер удержания по умолчанию: 15.

Выполним настройку интервалов:

--- R1 ----
```
interface s1/0
ip hello-interval eigrp 1 60
ip hold-time eigrp 1 180
interface s1/1
ip hello-interval eigrp 1 60
ip hold-time eigrp 1 180
```

--- R2 ----
```
interface s1/0
ip hello-interval eigrp 1 60
ip hold-time eigrp 1 180
interface s1/1
ip hello-interval eigrp 1 60
ip hold-time eigrp 1 180
```

--- R3 ----
```
interface s1/0
ip hello-interval eigrp 1 60
ip hold-time eigrp 1 180
interface s1/1
ip hello-interval eigrp 1 60
ip hold-time eigrp 1 180
```


Проверка:
```
R2(config)#do show ip eigrp interfaces detail
EIGRP-IPv4 Interfaces for AS(1)
                              Xmit Queue   PeerQ        Mean   Pacing Time   Multicast    Pending
Interface              Peers  Un/Reliable  Un/Reliable  SRTT   Un/Reliable   Flow Timer   Routes
Se1/1                    1        0/0       0/0           9       0/16          56           0
  Hello-interval is 60, Hold-time is 180
  Split-horizon is enabled
  Next xmit serial <none>
  Packetized sent/expedited: 16/0
  Hello's sent/expedited: 930/2
  Un/reliable mcasts: 0/0  Un/reliable ucasts: 18/17
  Mcast exceptions: 0  CR packets: 0  ACKs suppressed: 0
  Retransmissions sent: 0  Out-of-sequence rcvd: 0
  Topology-ids on interface - 0 
  Authentication mode is not set
```

#### Вопросы для повторения:
1. В чем заключаются преимущества объединения маршрутов?

Преимущество в уменьшении таблиц маршрутизации.

2. Почему при настройке таймеров EIGRP необходимо настраивать значение времени удержания равным или больше интервала приветствия?

В противном случае будет происходить разрыв соседства по таймауту.
