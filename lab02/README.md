# Настройка STP

### Топология
![](topologia.png)

### Таблица адресации
|Устройство|Интерфейс|IP-адрес     |Маска подсети|
|----------|---------|-------------|-------------|
|S1        |VLAN 1   |192.168.1.1  |255.255.255.0|
|S2        |VLAN 1   |192.168.1.2  |255.255.255.0|
|S3        |VLAN 1   |192.168.1.3  |255.255.255.0|

### Задачи
1. Создание сети и настройка основных параметров устройства
2. Выбор корневого моста
3. Наблюдение за процессом выбора протоколом STP порта, исходя из стоимости портов
4. Наблюдение за процессом выбора протоколом STP порта, исходя из приоритета портов

### Выполнение

#### 1. Создание сети и настройка основных параметров устройства
Выполняем базовую настройку на коммутаторах
```
conf t
hostname SX
no ip domain-lookup
enable secret class
line console 0
password cisco
exec-timeout 0 0
line vty 0 4
password cisco
exit

interface vlan 1
ip address 192.168.1.X 255.255.255.0
no shutdown
exit


service password-encryption
banner motd $ Authorized Access Only! $
```

Проверяем сетевую доступность между коммутаторами
```
S2#ping 192.168.1.3
Type escape sequence to abort.
Sending 5, 100-byte ICMP Echos to 192.168.1.3, timeout is 2 seconds:
.!!!!
Success rate is 80 percent (4/5), round-trip min/avg/max = 1/1/1 ms
S2#ping 192.168.1.1
Type escape sequence to abort.
Sending 5, 100-byte ICMP Echos to 192.168.1.1, timeout is 2 seconds:
.!!!!
Success rate is 80 percent (4/5), round-trip min/avg/max = 1/1/1 ms

```

#### 2. Выбор корневого моста
Выполняем настройки на коммутаторах
```
interface range ethernet 0/0-3  
shutdown
switchport trunk encapsulation dot1q
switchport mode trunk
exit
interface ethernet 0/1
no shutdown
exit
interface ethernet 0/3
no shutdown
exit

```

Результат show spanning-tree на S1
```
S1#show spanning-tree 

VLAN0001
  Spanning tree enabled protocol ieee
  Root ID    Priority    32769
             Address     aabb.cc00.1000
             This bridge is the root
             Hello Time   2 sec  Max Age 20 sec  Forward Delay 15 sec

  Bridge ID  Priority    32769  (priority 32768 sys-id-ext 1)
             Address     aabb.cc00.1000
             Hello Time   2 sec  Max Age 20 sec  Forward Delay 15 sec
             Aging Time  300 sec

Interface           Role Sts Cost      Prio.Nbr Type
------------------- ---- --- --------- -------- --------------------------------
Et0/1               Desg FWD 100       128.2    Shr 
Et0/3               Desg FWD 100       128.4    Shr 

```

Результат show spanning-tree на S2
```
S2#show spanning-tree 

VLAN0001
  Spanning tree enabled protocol ieee
  Root ID    Priority    32769
             Address     aabb.cc00.1000
             Cost        100
             Port        2 (Ethernet0/1)
             Hello Time   2 sec  Max Age 20 sec  Forward Delay 15 sec

  Bridge ID  Priority    32769  (priority 32768 sys-id-ext 1)
             Address     aabb.cc00.2000
             Hello Time   2 sec  Max Age 20 sec  Forward Delay 15 sec
             Aging Time  300 sec

Interface           Role Sts Cost      Prio.Nbr Type
------------------- ---- --- --------- -------- --------------------------------
Et0/1               Root FWD 100       128.2    Shr 
Et0/3               Desg FWD 100       128.4    Shr 

```

Результат show spanning-tree на S3
```
S3#show spanning-tree 

VLAN0001
  Spanning tree enabled protocol ieee
  Root ID    Priority    32769
             Address     aabb.cc00.1000
             Cost        100
             Port        4 (Ethernet0/3)
             Hello Time   2 sec  Max Age 20 sec  Forward Delay 15 sec

  Bridge ID  Priority    32769  (priority 32768 sys-id-ext 1)
             Address     aabb.cc00.3000
             Hello Time   2 sec  Max Age 20 sec  Forward Delay 15 sec
             Aging Time  300 sec

Interface           Role Sts Cost      Prio.Nbr Type
------------------- ---- --- --------- -------- --------------------------------
Et0/1               Altn BLK 100       128.2    Shr 
Et0/3               Root FWD 100       128.4    Shr 
```

Корневым мостом стал коммутатор S1, т.к. у него MAC-адрес с самым низким значением.

Корневые порты: S2 - Et0/1; S3 - Et0/3.
Назначенные порты: S1 - Et0/1, Et0/3; S2 - Et0/3.
Альтернативные порты: S3 - Et0/1.

#### 3. Наблюдение за процессом выбора протоколом STP порта, исходя из стоимости портов
