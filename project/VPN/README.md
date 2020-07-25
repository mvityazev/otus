# VPN. GRE. DmVPN

Необходимо настроить GRE между офисами Москва и С.-Петербург Настроить DMVPN между офисами Москва и Чокурдах, Лабытнанги.

Выполним настройку в следующей последовательности:
1. Настроим GRE между офисами Москва и С.-Петербург
2. Настроим DMVPN между Москва и Чокурдах, Лабытнанги

![](tunnel.png)

### 1. Настроим GRE между офисами Москва и С.-Петербург

Для обеспечения резервирования построим два туннеля между MSK и SPB.
Выполняем следующие настройки на маршрутизаторах:

Москва, AS 1001:

R14:
```
interface Tunnel1
description TUNNEL_SPB
ip address 10.1.255.29 255.255.255.252
ip mtu 1474
ip tcp adjust-mss 1456
tunnel source 200.1.0.126
tunnel destination 200.2.0.1
```

R15:
```
interface Tunnel2
ip address 10.1.255.33 255.255.255.252
ip mtu 1474
ip tcp adjust-mss 1456
tunnel source 200.1.0.254
tunnel destination 200.2.0.1
```

С-Петербург, AS 2042:

R18:
```
interface Tunnel1
description TUNNEL_MSK_R14
ip address 10.1.255.30 255.255.255.252
ip mtu 1474
ip tcp adjust-mss 1456
tunnel source 200.2.0.1
tunnel destination 200.1.0.126

interface Tunnel2
description TUNNEL_MSK_R15
ip address 10.1.255.34 255.255.255.252
ip mtu 1474
ip tcp adjust-mss 1456
tunnel source 200.2.0.1
tunnel destination 200.1.0.254
```

### 2. Настроим DMVPN между Москва и Чокурдах, Лабытнанги

Для обеспечения резервирования будем создавать по два туннеля до MSK. Настроим DMVPN в режиме работы Phase 3. Выполняем следующие настройки на маршрутизаторах:

Москва, AS 1001:

R14:
```
interface Tunnel10
 description DMVPN
 ip address 10.1.255.190 255.255.255.192
 no ip redirects
 ip nhrp authentication otus
 ip nhrp map multicast dynamic
 ip nhrp network-id 10
 ip nhrp holdtime 300
 ip nhrp redirect
 ip ospf network broadcast
 tunnel source 200.1.0.126
 tunnel mode gre multipoint
```

R15:
```
interface Tunnel10
 description DMVPN_REZ
 ip address 10.1.255.254 255.255.255.192
 no ip redirects
 ip nhrp authentication otus
 ip nhrp map multicast dynamic
 ip nhrp network-id 10
 ip nhrp holdtime 300
 ip nhrp redirect
 ip ospf network broadcast
 tunnel source 200.1.0.254
 tunnel mode gre multipoint
```

Лабытнанги (R27):
```
interface Tunnel0
 description DMVPN_MSK_R14
 ip address 10.1.255.129 255.255.255.192
 no ip redirects
 ip nhrp authentication otus
 ip nhrp map 10.1.255.190 200.1.0.126
 ip nhrp map multicast 200.1.0.126
 ip nhrp network-id 10
 ip nhrp holdtime 300
 ip nhrp nhs 10.1.255.190
 ip nhrp shortcut
 ip ospf network broadcast
 ip ospf priority 0
 tunnel source Ethernet0/0
 tunnel mode gre multipoint

interface Tunnel10
 description DMVPN_MSK_R15
 ip address 10.1.255.193 255.255.255.192
 no ip redirects
 ip nhrp authentication otus
 ip nhrp map 10.1.255.254 200.1.0.254
 ip nhrp map multicast 200.1.0.254
 ip nhrp network-id 10
 ip nhrp holdtime 300
 ip nhrp nhs 10.1.255.254
 ip nhrp shortcut
 ip ospf network broadcast
 ip ospf priority 0
 tunnel source Ethernet0/0
 tunnel mode gre multipoint
```

Чокурдах (R28):
```
interface Tunnel0
 description DMVPN_MSK_R14
 ip address 10.1.255.130 255.255.255.192
 no ip redirects
 ip nhrp authentication otus
 ip nhrp map 10.1.255.190 200.1.0.126
 ip nhrp map multicast 200.1.0.126
 ip nhrp network-id 10
 ip nhrp holdtime 300
 ip nhrp nhs 10.1.255.190
 ip nhrp shortcut
 ip ospf network broadcast
 ip ospf priority 0
 tunnel source Ethernet0/0
 tunnel mode gre multipoint

interface Tunnel10
 description DMVPN_MSK_R15
 ip address 10.1.255.194 255.255.255.192
 no ip redirects
 ip nhrp authentication otus
 ip nhrp map 10.1.255.254 200.1.0.254
 ip nhrp map multicast 200.1.0.254
 ip nhrp network-id 10
 ip nhrp holdtime 300
 ip nhrp nhs 10.1.255.254
 ip nhrp shortcut
 ip ospf network broadcast
 ip ospf priority 0
 tunnel source Ethernet0/1
 tunnel mode gre multipoint
```
