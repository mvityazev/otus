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
````
router rip
version 2
network 192.168.1.0
network 10.0.0.0
```
