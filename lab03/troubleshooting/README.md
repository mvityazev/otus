# Поиск и устранение неполадок в работе EtherChannel

### Топология
![](eve.png)

### Таблица адресации
|Устройство|Интерфейс|IP-адрес     |Маска подсети|
|----------|---------|-------------|-------------|
|S1        |VLAN 99  |192.168.1.11 |255.255.255.0|
|S2        |VLAN 99  |192.168.1.12 |255.255.255.0|
|S3        |VLAN 99  |192.168.1.13 |255.255.255.0|
|PC-A      |NIC      |192.168.0.1  |255.255.255.0|
|PC-C      |NIC      |192.168.0.3  |255.255.255.0|

### Задачи
Часть 1. Построение сети и загрузка настроек устройств

Часть 2. Отладка EtherChannel

### Выполнение

#### 1. Построение сети и загрузка настроек устройств

### 2. Поиск и устранение неисправностей в EtherChannel
Выполняем show interfaces trunk на коммутаторах:
```
S1#sh int trunk

Port        Mode             Encapsulation  Status        Native vlan
Et0/0       on               802.1q         trunking      99
Et0/1       on               802.1q         trunking      99
Po2         auto             802.1q         trunking      99

Port        Vlans allowed on trunk
Et0/0       none
Et0/1       none
Po2         1-4094

Port        Vlans allowed and active in management domain
Et0/0       none
Et0/1       none
Po2         1,10,99

Port        Vlans in spanning tree forwarding state and not pruned
Et0/0       none
Et0/1       none
Po2         1,10,99
```

```
S2#sh int trunk

Port        Mode             Encapsulation  Status        Native vlan
Et0/0       on               802.1q         trunking      99
Et0/1       on               802.1q         trunking      99

Port        Vlans allowed on trunk
Et0/0       1,99
Et0/1       1,99

Port        Vlans allowed and active in management domain
Et0/0       1,99
Et0/1       1,99

Port        Vlans in spanning tree forwarding state and not pruned
Et0/0       1,99
Et0/1       1,99
```

```
S3# sh int trunk

Port        Mode             Encapsulation  Status        Native vlan
Et0/1       on               802.1q         trunking      99
Po3         on               802.1q         trunking      99

Port        Vlans allowed on trunk
Et0/1       1-4094
Po3         1-4094

Port        Vlans allowed and active in management domain
Et0/1       1,10,99
Po3         1,10,99

Port        Vlans in spanning tree forwarding state and not pruned
Et0/1       1,10,99
Po3         1,10,99
```
