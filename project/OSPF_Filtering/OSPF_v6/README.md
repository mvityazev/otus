# Настройка OSPF для IPv6 в офисе Москва

Выполним настройку OSPF для IPv6 аналогично OSPF IPv4:

1. Маршрутизаторы R14-R15 находятся в зоне 0 - backbone

2. Маршрутизаторы R12-R13 находятся в зоне 10. Дополнительно к маршрутам должны получать маршрут по-умолчанию

3. Маршрутизатор R19 находится в зоне 101 и получает только маршрут по умолчанию

4. Маршрутизатор R20 находится в зоне 102 и получает все маршруты, кроме маршрутов до сетей зоны 101

 
![](moscow.png)
 

## 1. Настройка area 0


Выполним настройки OSPF:
R14:
```
ipv6 unicast-routing

ipv6 router ospf 1
passive-interface e0/2

int e0/0
ipv6 ospf 1 area 10
int e0/1
ipv6 ospf 1 area 10
int e0/3
ipv6 ospf 1 area 101
int e1/0
ipv6 ospf 1 area 0
int lo0
ipv6 ospf 1 area 0

```

R15:
```
ipv6 unicast-routing

ipv6 router ospf 1
passive-interface e0/2

int e0/0
ipv6 ospf 1 area 10
int e0/1
ipv6 ospf 1 area 10
int e0/3
ipv6 ospf 1 area 102
int e1/0
ipv6 ospf 1 area 0
int lo0
ipv6 ospf 1 area 0

```


## 2. Настройка area 10
В 10 зоне дополнительно к маршрутам требуется получать маршрут по-умолчанию, для этого определим эту зону как stub.

R12:
```
ipv6 unicast-routing
ipv6 router ospf 1
area 10 stub

int e0/0.7
ipv6 ospf 1 area 10
int e0/0.10
ipv6 ospf 1 area 10
int e0/0.99
ipv6 ospf 1 area 10
int e0/2
ipv6 ospf 1 area 10
int e0/3
ipv6 ospf 1 area 10
int lo0
ipv6 ospf 1 area 10
```


R13:
```
ipv6 unicast-routing
ipv6 router ospf 1
area 10 stub

int e0/0.7
ipv6 ospf 1 area 10
int e0/0.10
ipv6 ospf 1 area 10
int e0/0.99
ipv6 ospf 1 area 10
int e0/2
ipv6 ospf 1 area 10
int e0/3
ipv6 ospf 1 area 10
int lo0
ipv6 ospf 1 area 10
```

Также необходимо внести дополнительные настройки на R14 и R15:
```
ipv6 router ospf 1
area 10 stub
```

## 3. Настройка area 101
Маршрутизатор в 101 зоне должен получать только маршрут по-умолчанию, для этого определим эту зону как total stub.
R19:
```
ipv6 unicast-routing
ipv6 router ospf 1
area 101 stub no-summary

int e0/0
ipv6 ospf 1 area 101
int lo0
ipv6 ospf 1 area 101
```

Внесем изменения на R14 и R15:
```
ipv6 router ospf 1
area 101 stub no-summary
```

## 3. Настройка area 102
Маршрутизаторы 102 зоны должны получать все маршруты, кроме маршрутов 101 зоны. Для этого на ABR R15 настроим фильтрацию подсетей маршрутизатора R19.
```
ipv6 prefix-list FROM_R19_V6 seq 10 deny 2001:A:1::19/128
ipv6 prefix-list FROM_R19_V6 seq 20 deny 2001:A:1:1403/64 le 128
ipv6 prefix-list FROM_R19_V6 seq 30 permit ::/0 le 128

ipv6 router ospf 1
area 102 filter-list prefix FROM_R19_V6 in
```

Выполним настройки OSPF на R20:
```
ipv6 unicast-routing
ipv6 router ospf 1

int e0/0
ipv6 ospf 1 area 102
int lo0
ipv6 ospf 1 area 102
```
