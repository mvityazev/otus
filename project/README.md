#  IPv4/v6

###  Задание:

  1. Разработать и задокументировать адресное пространство;
  2. Настроить IP адреса на каждом активном порту;
  3. Использовать IPv4 и IPv6;
  4. Задокументировать все изменения.



###  Решение:

###  1. Задокументируем используемое адресное пространство с использованием IPv4 и IPv6.

| Устр-во | Интерфейс | IPv4             | Суммарная сеть IPv4 | IPv6 unicast           | IPv6 link-local | Суммарная сеть IPv6 |
|------------|-----------|------------------|---------------------|------------------------|-----------------|---------------------|
| R14        | E0/0      | 10.1.255.1/30    | 10.1.0.0/16         | 2001:A:1:1400::/64     | fe80::14/10     | 2001:A:1::/48       |
| R14        | E0/1      | 10.1.255.5/30    |                     | 2001:A:1:1401::/64     |                 |                     |
| R14        | E0/2      | 100.10.0.2/30    |                     | 2001:B:1:2200::1402/64 |                 |                     |
| R14        | E0/3      | 10.1.255.9/30    |                     | 2001:A:1:1403::/64     |                 |                     |
| R14        | Lo0       | 10.1.254.14/32   |                     | 2001:A:1::14/128       |                 |                     |
| R15        | E0/0      | 10.1.255.13/30   |                     | 2001:A:1:1500::/64     | Fe80::15/10     |                     |
| R15        | E0/1      | 10.1.255.17/30   |                     | 2001:A:1:1501::/64     |                 |                     |
| R15        | E0/2      | 100.20.0.2/30    |                     | 2001:B:2:2100::1502/64 |                 |                     |
| R15        | E0/3      | 10.1.255.21/30   |                     | 2001:A:1:1503::/64     |                 |                     |
| R15        | Lo0       | 10.1.254.15/24   |                     | 2001:A:1::15/128       |                 |                     |
| R19        | E0/0      | 10.1.255.10/30   |                     | 2001:A:1:1403::1900/64 | Fe80::19/10     |                     |
| R19        | Lo0       | 10.1.254.19/32   |                     | 2001:A:1::19/128       |                 |                     |
| R12        | E0/0      | 10.1.255.25/30   |                     | 2001:A:1:1200::/64     | Fe80::12/10     |                     |
| R12        | E0/2      | 10.1.255.2/30    |                     | 2001:A:1:1400::1202/64 |                 |                     |
| R12        | E0/3      | 10.1.255.18/30   |                     | 2001:A:1:1501::1203/64 |                 |                     |
| R12        | Lo0       | 10.1.254.12/24   |                     | 2001:A:1::12/128       |                 |                     |
| R12        | Vlan10    | 10.1.0.254/24    |                     | 2001:A:1:ff10::12/64   | Fe80::12/10     |                     |
| R12        | Vlan7     | 10.1.1.254/24    |                     | 2001:A:1:fff7::12/64   |                 |                     |
| R12        | Vlan99    | 10.1.254.213/25  |                     | 2001:A:1:ff99::12/64   |                 |                     |
| R13        | E0/0      | 10.1.255.29/30   |                     | 2001:A:1:1300::/64     | Fe80::13/10     |                     |
| R13        | E0/2      | 10.1.255.14/30   |                     | 2001:A:1:1503::1302/64 |                 |                     |
| R13        | E0/3      | 10.1.255.6/30    |                     | 2001:A:1:1401::1303/64 |                 |                     |
| R13        | Lo0       | 10.1.254.13/32   |                     | 2001:A:1::13/128       |                 |                     |
| R13        | Vlan10    | 10.1.0.253/24    |                     | 2001:A:1:ff10::13/64   | Fe80::13/10     |                     |
| R13        | Vlan7     | 10.1.1.254/24    |                     | 2001:A:1:fff7::13/64   |                 |                     |
| R13        | Vlan99    | 10.1.254.13/25   |                     | 2001:A:1:ff99::13/64   |                 |                     |
| R20        | E0/0      | 10.1.255.22/30   |                     | 2001:A:1:1403::2000/64 | Fe80::20/10     |                     |
| R20        | Lo0       | 10.1.254.20/32   |                     | 2001:A:1::20/128       |                 |                     |
| SW5        | Vlan99    | 10.1.254.215/25  |                     | 2001:A:1:ff99::5/64    | Fe80::5/10      |                     |
| SW4        | Vlan99    | 10.1.254.214/25  |                     | 2001:A:1:ff99::4/64    | Fe80::4/10      |                     |
| SW3        | Vlan99    | 10.1.254.213/25  |                     | 2001:A:1:ff99::3/64    | Fe80::3/10      |                     |
| SW2        | Vlan99    | 10.1.254.212/25  |                     | 2001:A:1:ff99::2/64    | Fe80::2/10      |                     |
| VPC1       | eth0      | 10.1.0.1/24      |                     | 2001:A:1:fff1::1/64    | Fe80::1/10      |                     |
| VPC7       | eth0      | 10.1.1.7/24      |                     | 2001:A:1:fff7::7/64    | Fe80::7/10      |                     |
|            |           |                  |                     |                        |                 |                     |
| R22        | E0/0      | 100.10.0.1/30    | 100.10.0.0/16       | 2001:B:1:2200::/64     | Fe80::22/10     | 2001:B:1:/48        |
| R22        | E0/1      | 100.20.0.6/30    |                     | 2001:B:2:2101::2201/64 | Fe80::22/10     |                     |
| R22        | E0/2      | 100.10.0.5/30    |                     | 2001:B:1:2202::/64     | Fe80::22/10     |                     |
| R22        | Lo0       | 100.10.255.22/32 |                     | 2001:B:1::22/128       |                 |                     |
|            |           |                  |                     |                        |                 |                     |
| R21        | E0/0      | 100.20.0.1/30    | 100.20.0.0/16       | 2001:B:2:2100::/64     | Fe80::21/10     | 2001:B:2:/48        |
| R21        | E/0/1     | 100.20.0.5/30    |                     | 2001:B:2:2101::/64     | Fe80::21/10     |                     |
| R21        | E/0/2     | 100.30.0.2/30    |                     | 2001:B:2:2102::/64     | Fe80::21/10     |                     |
| R21        | Lo0       | 100.20.255.21/32 |                     | 2001:B:2::21/128       |                 |                     |
|            |           |                  |                     |                        |                 |                     |
| R23        | E0/0      | 100.10.0.6/30    | 100.30.0.0/16       | 2001:B:1:2202::2300/64 | Fe80::23/10     | 2001:B:3:/48        |
| R23        | E0/1      | 100.30.0.6/30    |                     | 2001:B:3:2301::/64     | Fe80::23/10     |                     |
| R23        | E0/2      | 100.30.0.9/30    |                     | 2001:B:3:2302::/64     | Fe80::23/10     |                     |
| R23        | Lo0       | 10.30.255.23/32  |                     | 2001:B:3::23/128       |                 |                     |
| R24        | E0/0      | 100.30.0.1/32    |                     | 2001:B:2:2102::2400/64 | Fe80::24/10     |                     |
| R24        | E0/1      | 100.30.0.14/30   |                     | 2001:B:3:2600::2401/64 | Fe80::24/10     |                     |
| R24        | E0/2      | 100.30.0.10/30   |                     | 2001:B:3:2302::2402/64 | Fe80::24/10     |                     |
| R24        | E0/3      | 100.30.0.17/30   |                     | 2001:B:3:2403::/64     | Fe80::24/10     |                     |
| R24        | Lo0       | 100.30.255.24/32 |                     | 2001:B:3::24/128       |                 |                     |
| R25        | E0/0      | 100.30.0.5/30    |                     | 2001:B:3:2301::2500/64 | Fe80::25/10     |                     |
| R25        | E0/1      | 100.30.0.21/30   |                     | 2001:B:3:2501::/64     | Fe80::25/10     |                     |
| R25        | E0/2      | 100.30.0.25/30   |                     | 2001:B:3:2502::/64     | Fe80::25/10     |                     |
| R25        | E0/3      | 100.30.0.29/30   |                     | 2001:B:3:2503::/64     | Fe80::25/10     |                     |
| R25        | Lo0       | 100.30.255.25/32 |                     | 2001:B:3::25/128       |                 |                     |
| R26        | E0/0      | 100.30.0.13/32   |                     | 2001:B:3:2600::/64     | Fe80::26/10     |                     |
| R26        | E0/1      | 100.30.0.34/32   |                     | 2001:B:3:2601::/64     | Fe80::26/10     |                     |
| R26        | E0/2      | 100.30.0.26/30   |                     | 2001:B:3:2502::2602/64 | Fe80::26/10     |                     |
| R26        | E0/3      | 100.30.0.38/30   |                     | 2001:B:3:2603::/64     | Fe80::26/10     |                     |
| R26        | Lo0       | 100.30.255.26/32 |                     | 2001:B:3::26/128       |                 |                     |
|            |           |                  |                     |                        |                 |                     |
| R27        | E0/0      | 100.30.0.22/30   | 10.3.2.0/23         | 2001:B:3:2501::2700/64 | Fe80::27/10     | 2001:A:4::/48       |
| R27        | Lo0       | 10.3.2.129/32    |                     | 2001:A:4::27/128       |                 |                     |
|            |           |                  |                     |                        |                 |                     |
| R28        | E0/0      | 100.30.0.33/30   | 10.3.0.0/23         | 2001:B:3:2601::2800/64 | Fe80::28/10     | 2001:A:3::/48       |
| R28        | E0/1      | 100.30.0.30/30   |                     | 2001:B:3:2503::2801/64 | Fe80::28/10     |                     |
| R28        | Lo0       | 10.3.1.129/32    |                     | 2001:A:3::28/128       |                 |                     |
| R28        | Vlan30    | 10.3.0.62/26     |                     | 2001:A:3:ff30::28/64   | Fe80::28/10     |                     |
| R28        | Vlan31    | 10.3.0.126/26    |                     | 2001:A:3:ff31::28/64   | Fe80::28/10     |                     |
| R28        | Vlan99    | 10.3.1.161/27    |                     | 2001:A:3:ff99::28/64   | Fe80::28/10     |                     |
| SW29       | Vlan99    | 10.3.1.162/27    |                     | 2001:A:3:ff99::29/64   | Fe80::29/10     |                     |
| VPC30      | eth0      | 10.3.0.1/26      |                     | 2001:A:3:ff30::30/64   | Fe80::30/10     |                     |
| VPC31      | eth0      | 10.3.0.65/26     |                     | 2001:A:3:ff31::31/64   | Fe80::31/10     |                     |
|            |           |                  |                     |                        |                 |                     |
| R18        | E0/0      | 10.2.255.9/30    | 10.2.0.0/16         | 2001:A:2:1800::/64     | Fe80::18/10     | 2001:A:2::/48       |
| R18        | E0/1      | 10.2.255.5/30    |                     | 2001:A:2:1801::/64     | Fe80::18/10     |                     |
| R18        | E0/2      | 100.30.0.18/30   |                     | 2001:B:3:2403::1802/64 | Fe80::18/10     |                     |
| R18        | E0/3      | 100.30.0.38/30   |                     | 2001:B:3:2603::1803/64 | Fe80::18/10     |                     |
| R18        | Lo0       | 10.2.254.18/32   |                     | 2001:A:2::18/128       |                 |                     |
| R17        | E0/1      | 10.2.255.6/30    |                     | 2001:A:2:1801::1701/64 | Fe80::17/10     |                     |
| R17        | Lo0       | 10.2.254.17/32   |                     | 2001:A:2::17/128       |                 |                     |
| R17        | Vlan8     | 10.2.0.17/24     |                     | 2001:A:2:fff8::17/64   | Fe80::17/10     |                     |
| R17        | Vlan9     | 10.2.1.17/24     |                     | 2001:A:2:fff9::17/64   | Fe80::17/10     |                     |
| R17        | Vlan99    | 10.2.254.217/25  |                     | 2001:A:2:ff99::17/64   | Fe80::17/10     |                     |
| R16        | E0/0      | 10.2.255.21/30   |                     | 2001:A:2:1600::/64     | Fe80::16/10     |                     |
| R16        | E0/1      | 10.2.255.10/30   |                     | 2001:A:2:1800::1601/64 | Fe80::16/10     |                     |
| R16        | E0/3      | 10.2.255.13/30   |                     | 2001:A:2:1603::/64     | Fe80::16/10     |                     |
| R16        | Lo0       | 10.2.254.16/32   |                     | 2001:A:2::16/128       |                 |                     |
| R16        | Vlan8     | 10.2.0.16/24     |                     | 2001:A:2:fff8::16/64   | Fe80::16/10     |                     |
| R16        | Vlan9     | 10.2.1.16/24     |                     | 2001:A:2:fff9::16/64   | Fe80::16/10     |                     |
| R16        | Vlan99    | 10.2.254.216/25  |                     | 2001:A:2:ff99::16/64   | Fe80::16/10     |                     |
| R32        | E0/0      | 10.2.255.14/30   |                     | 2001:A:2:1603::3200/64 | Fe80::32/10     |                     |
| R32        | Lo0       | 10.2.254.32/32   |                     | 2001:A:2::32/128       |                 |                     |
| SW9        | Vlan99    | 10.2.254.219/25  |                     | 2001:A:2:ff99::9/64    | Fe80::9/10      |                     |
| SW10       | Vlan99    | 10.2.254.220/25  |                     | 2001:A:2:ff99::10/64   | Fe80::10/10     |                     |
| VPC8       | eth0      | 10.2.0.1/24      |                     | 2001:A:2:fff8::8/64    | Fe80::8/10      |                     |
| VPC9       | eth0      | 10.2.1.1/24      |                     | 2001:A:2:fff9::9/64    | Fe80::9/10      |                     |