
Третья лабораторная работа
# IS-IS Underlay в сети CLOS

- [IS-IS Underlay в сети CLOS](#is-is-underlay-в-сети-clos)
  - [1. Базовые настройки IS-IS для Cisco Nexus v9500.](#1-базовые-настройки-is-is-для-cisco-nexus-v9500)
  - [2. Проверка работы IS-IS](#2-проверка-работы-is-is)
  - [3. Итоговые конфигурации устройств:](#3-итоговые-конфигурации-устройств)

***
[Таблица IP адресов из первой лабораторной работы](https://github.com/AndreyNechaevGit/DCNetwork/blob/main/lab01/README.md#%D1%82%D0%B0%D0%B1%D0%BB%D0%B8%D1%86%D0%B0-ip-%D0%B0%D0%B4%D1%80%D0%B5%D1%81%D0%BE%D0%B2)
***

**Цели:**
- Настроить IS-IS для Underlay в сети CLOS;

**В этой самостоятельной работе мы ожидаем, что вы самостоятельно:**
- настроите IS-IS в Underlay сети, для IP связности между всеми устройствами NXOS
- составите план работы, адресное пространство, схему сети, настройки

 **Реализовать схему IS-IS on UNDERLAY** по схеме:
![image](https://user-images.githubusercontent.com/60564360/213522890-5f0c858e-f21d-4c94-91cc-652493228752.png)


##  1. Базовые настройки IS-IS для Cisco Nexus v9500.

На примере **Spline01**

Предварительно на всех устройствах настроены необходимые интерфейсы и IP адресация.

```
interface Ethernet1/1
  description to-leaf-01
  ip address 10.10.1.0/31
  no switchport
  no shutdown
...
...
interface loopback1
  description for-underlay
  ip address 10.8.1.0/32
```

**1.1 Включим протокол IS-IS и выполним базовую настройку**
  
```
feature isis

router isis UNDERLAY
  net 49.0011.0001.0001.0002.00
  is-type level-2
  log-adjacency-changes
  passive-interface default level-2
```

* `router isis Underlay` - указание строкового параметра в качестве ID IS-IS процесса
* `net` - идентификатор для устройств в протоколе isis должен быть уникальным для каждого устройства в области  The `NET ID` is comprised of the IS-IS system ID, which uniquely identifies this IS-IS instance in the area, and the area ID. For example, if the NET ID is **49.0011**.*0008.0001.0000.00*, the `System ID` is *0008.0001.0000.00* and the `area ID` is **49.0011**.
> для каждого устройства укажем идентификатор net 49.0011.0001.000x.000y.00 y согласно lo1 10.8.x.y <br>
> Lo1 10.8.2.0 <br>
> NET 49.0010.0008.0002.0000.00
* `is-type level-2` - указываем для использования протоколом IS-IS только L2 соседства
* `log-adjacency-changes` - используется для отлеживания в логах состояние IS-IS соседства
* `passive-interface default level-2` - указание не использовать все интерфейсы в ISIS

**1.2 Донастроим P2P интерфейсы на всех свитчах:**

```
conf t
interface Ethernet1/1-3
  medium p2p
  isis network point-to-point
  isis circuit-type level-2
  ip router isis UNDERLAY
  no shutdown
```

**1.3 Добавим lo1 интерфейсы на всех свитчах в OSPF:**

```
conf t
interface loopback1
  ip router isis Underlay
```

* `ip router isis Underlay` - укажем использовать IS-IS на нужных интефейсах
* `isis network point-to-point` - в протоколе ISIS можно использовать два типа сетей Broadcast и Point-to-Point. Broadcast используется по умолчанию. Тип нужно указывать на дух сторонах линка
* `medium p2p` - Configures the interface medium as either point to point


**1.4 После проверки, что все работатет, дополнительно настроим авторизацию:**

На всех устройствах выполним:

Для настройки аутификации MD5 зоны
```
conf t

key chain ISIS
  key 0
    key-string 7 0758744a1d5b480643105e09

router isis UNDERLAY
  authentication-type md5 level-2
  authentication key-chain ISIS level-2
```

На интерфейсах участвующих в ISIS, также настроим авторизацию MD5

например для всех Leaf 

```
conf t

interface Ethernet1/6-7
  isis authentication-type md5
  isis authentication key-chain ISIS
```

***

##  2. Проверка работы IS-IS

После того как все устройства будут настроены, можно проверить статус протокола

Например на **Spine01**: 

```
Spine01# show isis adjacency
IS-IS process: UNDERLAY VRF: default
IS-IS adjacency database:
Legend: '!': No AF level connectivity in given topology
System ID       SNPA            Level  State  Hold Time  Interface
Leaf01          N/A             2      UP     00:00:26   Ethernet1/1
Leaf02          N/A             2      UP     00:00:31   Ethernet1/2
Leaf03          N/A             2      UP     00:00:25   Ethernet1/3

Spine01# show isis interface brief
IS-IS process: UNDERLAY VRF: default
Interface    Type  Idx State        Circuit   MTU  Metric  Priority  Adjs/AdjsUp
                                                   L1  L2  L1  L2    L1    L2
--------------------------------------------------------------------------------
Topology: TopoID: 0
loopback1    Loop  4     Up/Ready   0x01/L2   1500 1   1   64  64    0/0   0/0
Topology: TopoID: 0
Ethernet1/1  P2P   1     Up/Ready   0x01/L2   1500 40  40  64  64    0/0   1/1
Topology: TopoID: 0
Ethernet1/2  P2P   2     Up/Ready   0x01/L2   1500 40  40  64  64    0/0   1/1
Topology: TopoID: 0
Ethernet1/3  P2P   3     Up/Ready   0x01/L2   1500 40  40  64  64    0/0   1/1


Spine01# show isis topology base
IS-IS process: UNDERLAY
VRF: default
Topology ID: 0

IS-IS Level-1 IS routing table

IS-IS Level-2 IS routing table
Leaf01.00, Instance 0x0000002C
   *via Leaf01, Ethernet1/1, metric 40
Leaf02.00, Instance 0x0000002C
   *via Leaf02, Ethernet1/2, metric 40
Leaf03.00, Instance 0x0000002C
   *via Leaf03, Ethernet1/3, metric 40
Spine02.00, Instance 0x0000002C
   *via Leaf01, Ethernet1/1, metric 80
   *via Leaf02, Ethernet1/2, metric 80
   *via Leaf03, Ethernet1/3, metric 80

Spine01# show routing isis-Underlay
IP Route Table for VRF "default"
'*' denotes best ucast next-hop
'**' denotes best mcast next-hop
'[x/y]' denotes [preference/metric]
'%<string>' in via output denotes VRF <string>

10.8.0.1/32, ubest/mbest: 1/0
    *via 10.10.1.1, Eth1/1, [115/41], 00:48:33, isis-UNDERLAY, L2
10.8.0.2/32, ubest/mbest: 1/0
    *via 10.10.1.3, Eth1/2, [115/41], 00:46:31, isis-UNDERLAY, L2
10.8.0.3/32, ubest/mbest: 1/0
    *via 10.10.1.5, Eth1/3, [115/41], 00:46:45, isis-UNDERLAY, L2
10.8.2.0/32, ubest/mbest: 3/0
    *via 10.10.1.1, Eth1/1, [115/81], 00:48:27, isis-UNDERLAY, L2
    *via 10.10.1.3, Eth1/2, [115/81], 00:41:25, isis-UNDERLAY, L2
    *via 10.10.1.5, Eth1/3, [115/81], 00:41:24, isis-UNDERLAY, L2
10.10.2.0/31, ubest/mbest: 1/0
    *via 10.10.1.1, Eth1/1, [115/80], 00:48:33, isis-UNDERLAY, L2
10.10.2.2/31, ubest/mbest: 1/0
    *via 10.10.1.3, Eth1/2, [115/80], 00:46:31, isis-UNDERLAY, L2
10.10.2.4/31, ubest/mbest: 1/0
    *via 10.10.1.5, Eth1/3, [115/80], 00:46:45, isis-UNDERLAY, L2
```

<details>
<summary> Мини дебаг ISIS: </summary>

```
clear isis adjacency *
restart isis UNDERLAY
show isis event-history adjacency
show isis internal event-history msgs
show isis internal event-history errors
show running-config isis all
show routing unicast clients
show ip arp
sho cdp nei

show ip traffic
show processes memory | include isis
show isis traffic

show isis Underlay ethernet1/2

```
</details>
<br>


И выполнить запросы между несвязанными напрямую устройствами. <br>
Например между свитчем **Spine01** и **Spine02**
```
Spine01# ping 10.8.2.0 source 10.8.1.0
PING 10.8.2.0 (10.8.2.0) from 10.8.1.0: 56 data bytes
64 bytes from 10.8.2.0: icmp_seq=0 ttl=253 time=7.952 ms
64 bytes from 10.8.2.0: icmp_seq=1 ttl=253 time=4.971 ms
64 bytes from 10.8.2.0: icmp_seq=2 ttl=253 time=3.855 ms
64 bytes from 10.8.2.0: icmp_seq=3 ttl=253 time=3.684 ms
64 bytes from 10.8.2.0: icmp_seq=4 ttl=253 time=5.269 ms

--- 10.8.2.0 ping statistics ---
5 packets transmitted, 5 packets received, 0.00% packet loss
round-trip min/avg/max = 3.684/5.146/7.952 ms


Spine01# traceroute 10.8.2.0 source 10.8.1.0
traceroute to 10.8.2.0 (10.8.2.0) from 10.8.1.0 (10.8.1.0), 30 hops max, 40 byte packets
 1  10.10.1.5 (10.10.1.5)  20.507 ms  3.277 ms  2.26 ms
 2  10.8.2.0 (10.8.2.0)  23.477 ms  8.103 ms  4.499 ms
```

Проверим таблицу маршрутов с **Spine01** и **Spine02** , видно что выбраны 3 маршрута для задействования ECMP
```
Spine01# show ip route 10.8.2.0
IP Route Table for VRF "default"
'*' denotes best ucast next-hop
'**' denotes best mcast next-hop
'[x/y]' denotes [preference/metric]
'%<string>' in via output denotes VRF <string>

10.8.2.0/32, ubest/mbest: 3/0
    *via 10.10.1.1, Eth1/1, [115/81], 00:49:10, isis-UNDERLAY, L2
    *via 10.10.1.3, Eth1/2, [115/81], 00:42:08, isis-UNDERLAY, L2
    *via 10.10.1.5, Eth1/3, [115/81], 00:42:07, isis-UNDERLAY, L2

```


##  3. Итоговые конфигурации устройств:

`show run | section interface|router|key|feature|hostname`

**Spine01**

```
hostname Spine01

feature isis


key chain ISIS
  key 0
    key-string 7 0758744a1d5b480643105e09

interface Ethernet1/1
  description to-leaf-01
  medium p2p
  ip address 10.10.1.0/31
  isis network point-to-point
  isis circuit-type level-2
  isis authentication-type md5
  isis authentication key-chain ISIS
  ip router isis UNDERLAY
  no isis passive-interface level-2
  no shutdown

interface Ethernet1/2
  description to-leaf-02
  medium p2p
  ip address 10.10.1.2/31
  isis network point-to-point
  isis circuit-type level-2
  isis authentication-type md5
  isis authentication key-chain ISIS
  ip router isis UNDERLAY
  no isis passive-interface level-2
  no shutdown

interface Ethernet1/3
  description to-leaf-03
  medium p2p
  ip address 10.10.1.4/31
  isis network point-to-point
  isis circuit-type level-2
  isis authentication-type md5
  isis authentication key-chain ISIS
  ip router isis UNDERLAY
  no isis passive-interface level-2
  no shutdown

interface loopback1
  ip address 10.8.1.0/32
  ip router isis UNDERLAY
router isis UNDERLAY
  net 49.0011.0008.0001.0000.00
  is-type level-2
  log-adjacency-changes
  authentication-type md5 level-2
  authentication key-chain ISIS level-2
  passive-interface default level-2

```

**Spine02**

```
hostname Spine02

feature isis

key chain ISIS
  key 0
    key-string 7 0758744a1d5b480643105e09

interface Ethernet1/1
  description to-leaf-01
  medium p2p
  ip address 10.10.2.0/31
  isis network point-to-point
  isis circuit-type level-2
  isis authentication-type md5
  isis authentication key-chain ISIS
  ip router isis UNDERLAY
  no isis passive-interface level-2
  no shutdown

interface Ethernet1/2
  description to-leaf-02
  medium p2p
  ip address 10.10.2.2/31
  isis network point-to-point
  isis circuit-type level-2
  isis authentication-type md5
  isis authentication key-chain ISIS
  ip router isis UNDERLAY
  no isis passive-interface level-2
  no shutdown

interface Ethernet1/3
  description to-leaf-03
  medium p2p
  ip address 10.10.2.4/31
  isis network point-to-point
  isis circuit-type level-2
  isis authentication-type md5
  isis authentication key-chain ISIS
  ip router isis UNDERLAY
  no isis passive-interface level-2
  no shutdown

interface loopback1
  ip address 10.8.2.0/32
  ip router isis UNDERLAY

router isis UNDERLAY
  net 49.0011.0008.0002.0000.00
  is-type level-2
  log-adjacency-changes
  authentication-type md5 level-2
  authentication key-chain ISIS level-2
  passive-interface default level-2

```
***
**Leaf01**

```
hostname Leaf01

feature isis

key chain ISIS
  key 0
    key-string 7 0758744a1d5b480643105e09

interface Ethernet1/1
  description to-PC1
  ip address 10.12.1.254/24

interface Ethernet1/6
  description to-spine-01
  medium p2p
  ip address 10.10.1.1/31
  isis network point-to-point
  isis circuit-type level-2
  isis authentication-type md5
  isis authentication key-chain ISIS
  ip router isis UNDERLAY
  no isis passive-interface level-2
  no shutdown

interface Ethernet1/7
  description to-spine-02
  medium p2p
  ip address 10.10.2.1/31
  isis network point-to-point
  isis circuit-type level-2
  isis authentication-type md5
  isis authentication key-chain ISIS
  ip router isis UNDERLAY
  no isis passive-interface level-2
  no shutdown


interface loopback1
  ip address 10.8.0.1/32
  ip router isis UNDERLAY

router isis UNDERLAY
  net 49.0011.0008.0000.0001.00
  is-type level-2
  log-adjacency-changes
  authentication-type md5 level-2
  authentication key-chain ISIS level-2
  passive-interface default level-2
```

**Leaf02**

```
hostname Leaf02

feature isis

key chain ISIS
  key 0
    key-string 7 0758744a1d5b480643105e09

interface Ethernet1/1
  description to-PC2
  ip address 10.12.2.254/24
  no shutdown

interface Ethernet1/6
  description to-spine-01
  medium p2p
  ip address 10.10.1.3/31
  isis network point-to-point
  isis circuit-type level-2
  isis authentication-type md5
  isis authentication key-chain ISIS
  ip router isis UNDERLAY
  no isis passive-interface level-2
  no shutdown

interface Ethernet1/7
  description to-spine-02
  medium p2p
  ip address 10.10.2.3/31
  isis network point-to-point
  isis circuit-type level-2
  isis authentication-type md5
  isis authentication key-chain ISIS
  ip router isis UNDERLAY
  no isis passive-interface level-2
  no shutdown

interface loopback1
  ip address 10.8.0.2/32
  ip router isis UNDERLAY

router isis UNDERLAY
  net 49.0011.0008.0000.0002.00
  is-type level-2
  log-adjacency-changes
  authentication-type md5 level-2
  authentication key-chain ISIS level-2
  passive-interface default level-2
```
**Leaf03**

```
hostname Leaf03

feature isis

key chain ISIS
  key 0
    key-string 7 0758744a1d5b480643105e09

interface Ethernet1/1
  description to-PC3
  ip address 10.12.3.254/24
  no shutdown

interface Ethernet1/2
  description to-PC4
  ip address 10.12.4.254/24
  no shutdown

interface Ethernet1/6
  description to-spine-01
  medium p2p
  ip address 10.10.1.5/31
  isis network point-to-point
  isis circuit-type level-2
  isis authentication-type md5
  isis authentication key-chain ISIS
  ip router isis UNDERLAY
  no isis passive-interface level-2
  no shutdown

interface Ethernet1/7
  description to-spine-02
  medium p2p
  ip address 10.10.2.5/31
  isis network point-to-point
  isis circuit-type level-2
  isis authentication-type md5
  isis authentication key-chain ISIS
  ip router isis UNDERLAY
  no isis passive-interface level-2
  no shutdown

interface loopback1
  description for-UNDERLAY
  ip address 10.8.0.3/32
  ip router isis UNDERLAY

router isis UNDERLAY
  net 49.0011.0008.0000.0003.00
  is-type level-2
  log-adjacency-changes
  authentication-type md5 level-2
  authentication key-chain ISIS level-2
  passive-interface default level-2
```
