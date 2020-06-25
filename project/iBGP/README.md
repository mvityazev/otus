# Настройка iBGP

Необходимо настроить iBGP в офисе Москва, настроить iBGP в сети провайдера Триада. Организовать полную IP связанность всех сетей.

![](../BGP/bgp.png)

Выполним настройку в следующей последовательности:
1. Настроим iBGP в офисом Москва между маршрутизаторами R14 и R15
2. Настроим iBGP в провайдере Триада
3. Выполним настройки в офисе Москва так, чтобы приоритетным провайдером стал Ламас
4. Настроим в офисе С.-Петербург протокол iBGP. (без использования протокола OSPF)
5. Выполним настройки офиса С.-Петербург так, чтобы трафик до любого офиса распределялся по двум линкам одновременно

### 1. Настроим iBGP в офисом Москва между маршрутизаторами R14 и R15

Выполняем следующие настройки на маршрутизаторах:

Москва, AS 1001 (R14):
```
router bgp 1001
neighbor 10.1.254.15 remote-as 1001
neighbor 10.1.254.15 update-source lo0
neighbor 10.1.254.15 next-hop-self
```

Москва, AS 1001 (R15):
```
router bgp 1001
neighbor 10.1.254.14 remote-as 1001
neighbor 10.1.254.14 update-source lo0
neighbor 10.1.254.14 next-hop-self
```

### 2. Настроим iBGP в провайдере Триада

Выполняем следующие настройки на маршрутизаторах:

Триада, AS 520 (R23). Настроим R23 в роли Router reflector.
```
router bgp 520
network 100.30.0.0 mask 255.255.0.0
neighbor 100.10.0.5 remote-as 101
neighbor 100.30.255.24 remote-as 520
neighbor 100.30.255.24 update-source lo0
neighbor 100.30.255.24 route-reflector-client
neighbor 100.30.255.25 remote-as 520
neighbor 100.30.255.25 update-source lo0
neighbor 100.30.255.25 route-reflector-client
neighbor 100.30.255.26 remote-as 520
neighbor 100.30.255.26 update-source lo0
neighbor 100.30.255.26 route-reflector-client
```

Триада, AS 520 (R24, R25, R26):
```
router bgp 520
neighbor 100.30.255.23 remote-as 520
neighbor 100.30.255.23 update-source lo0
```

### 3. Выполним настройки в офисе Москва так, чтобы приоритетным провайдером стал Ламас

Настроим local-preference на R15:
```
router bgp 1001
neighbor 100.20.0.1 route-map LAMAS in

route-map LAMAS permit 10
set local-preference 200
```

### 4. Настроим в офисе С.-Петербург протокол iBGP. (без использования протокола OSPF)

Выполняем следующие настройки на маршрутизаторах:

С-Петербург, AS 2042 (R18):
```
ip route 10.2.254.17 255.255.255.255 10.2.255.6
ip route 10.2.254.16 255.255.255.255 10.2.255.10
ip route 10.2.254.32 255.255.255.255 10.2.255.10
ip route 10.2.255.12 255.255.255.252 10.2.255.10
router bgp 2042
network 100.30.0.18 mask 255.255.255.255
network 100.30.0.38 mask 255.255.255.255
neighbor 10.2.254.17 remote-as 2042
neighbor 10.2.254.17 update-source lo0
neighbor 10.2.254.17 next-hop-self
neighbor 10.2.254.16 remote-as 2042
neighbor 10.2.254.16 update-source lo0
neighbor 10.2.254.16 next-hop-self
neighbor 10.2.254.32 remote-as 2042
neighbor 10.2.254.32 update-source lo0
neighbor 10.2.254.32 next-hop-self
```

С-Петербург, AS 2042 (R17):
```
ip route 10.2.254.18 255.255.255.255 10.2.255.5
router bgp 2042
neighbor 10.2.254.18 remote-as 2042
neighbor 10.2.254.18 update-source lo0
neighbor 10.2.254.18 next-hop-self
```

С-Петербург, AS 2042 (R16):
```
ip route 10.2.254.18 255.255.255.255 10.2.255.9
ip route 10.2.254.32 255.255.255.255 10.2.255.14
router bgp 2042
neighbor 10.2.254.18 remote-as 2042
neighbor 10.2.254.18 update-source lo0
neighbor 10.2.254.18 next-hop-self
```

С-Петербург, AS 2042 (R32):
```
ip route 10.2.254.18 255.255.255.255 10.2.255.13
router bgp 2042
neighbor 10.2.254.18 remote-as 2042
neighbor 10.2.254.18 update-source lo0
neighbor 10.2.254.18 next-hop-self
```

### 5. Выполним настройки офиса С.-Петербург так, чтобы трафик до любого офиса распределялся по двум линкам одновременно


Выполняем следующие настройки на маршрутизаторах:

С-Петербург, AS 2042 (R18):
```
router bgp 2042
maximum-paths 2
```
