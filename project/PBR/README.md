=Политика маршрутизации в офисе Чокурдах =

==Настройки в офисе Чокурдах==

В офисе Чокурдах имеется два подключения к провайдеру Триада. Распределим трафик между 2 линками. 
Трафик с подсеть 10.3.0.0/26 будем отправлять через интерфейс e0/1, если данный канал не доступен, то через e0/0;
остальной трафик будем отправлять через интерфейс e0/0, если данных канал не доступен, то через e0/1.

Для распределения трафика между двумя линками с провайдером в Чокурдах выполним настройку на маршрутизаторе R28:

Пропишем два маршрута по умолчанию, но с разной административной дистанцией:
```
ip route 0.0.0.0 0.0.0.0 100.30.0.29 50
ip route 0.0.0.0 0.0.0.0 100.30.0.34 100
ipv6 route ::/0 2001:B:3:2503::1 50
ipv6 route ::/0 2001:B:3:2601::1 100
```

Выполним настройку, чтобы трафик от подсети 10.3.0.0/26 отправлялся через интерфейс e0/1:
Для IPv4:
```
ip access-list standart WAN1
permit 10.3.0.0 0.0.0.63
deny any
exit

route-map WAN1 permit 10
match ip address WAN1
set ip next-hop 100.30.0.34

int e0/2.30
ip policy route-map WAN1

```

Для IPv6:

```
ipv6 access-list WAN1_IP6
permit ipv6 2001:A:3:ff30::/64 any
deny any any
exit

route-map WAN1_IP6 permit 10
match ipv6 address WAN1_IP6
set ipv6 next-hop 2001:B:3:2601::1

int e0/2.30
ipv6 policy route-map WAN1_IP6
```

=== Настроим IP SLA ===

Для IPv4:
```
ip sla 1
icmp-jitter 100.30.0.34 source-ip 100.30.0.33 num-packets 5
frequency 10
ip sla schedule 1 life forever start-time now

track 1 ip sla 1 reachability

route-map WAN1 permit 10
no set ip next-hop 100.30.0.34
set ip next-hop verify-availability 100.30.0.34 10 track 1
set ip next-hop 100.30.0.29 20
```

Для IPv6 надоступна опция verify-availability в route-map, поэтому для него IP SLA не настраиваем.


== Настройки для офиса Лабытнанги ==
Лабытнанги имеет одно подключение к провайдеру Триада. 
Настроим маршрут по умолчанию на маршрутизаторе R27:
```
ip route 0.0.0.0 0.0.0.0 100.30.0.21
ipv6 route ::/0 2001:B:3:2501::1
```
