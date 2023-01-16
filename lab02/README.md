# Вторая лабораторная работа

[Таблица IP адресов из первой лабораторной работы](https://github.com/AndreyNechaevGit/DCNetwork/blob/main/lab01/README.md#%D1%82%D0%B0%D0%B1%D0%BB%D0%B8%D1%86%D0%B0-ip-%D0%B0%D0%B4%D1%80%D0%B5%D1%81%D0%BE%D0%B2)

[Итоговая конфигурация устройств:](https://github.com/AndreyNechaevGit/DCNetwork/tree/main/lab02#%D0%B8%D1%82%D0%BE%D0%B3%D0%BE%D0%B2%D1%8B%D0%B5-%D0%BA%D0%BE%D0%BD%D1%84%D0%B8%D0%B3%D1%83%D1%80%D0%B0%D1%86%D0%B8%D0%B8-%D1%83%D1%81%D1%82%D1%80%D0%BE%D0%B9%D1%81%D1%82%D0%B2)

Вторая лабораторная работа
# OSPFv2 Underlay в сети CLOS



  - [1. Введение](#1-введение)
  - [2. Базовые настройки OSPFv2 для Cisco Nexus v9500.](#4-1-базовые-настройки-ospfv2-для-cisco-nexus-v9500)
    - [2.1. Проверка параметров работы OSPFv2:](#41-проверка-параметров-работы-ospfv2)
  - [3. Итоговые конфигурации устройств:](#8-итоговые-конфигурации-устройств)


[Таблица IP адресов из первой лабораторной работы](https://github.com/AndreyNechaevGit/DCNetwork/blob/main/lab01/README.md#%D1%82%D0%B0%D0%B1%D0%BB%D0%B8%D1%86%D0%B0-ip-%D0%B0%D0%B4%D1%80%D0%B5%D1%81%D0%BE%D0%B2)

[Итоговая конфигурация устройств:](https://github.com/AndreyNechaevGit/DCNetwork/tree/main/lab01#%D0%B8%D1%82%D0%BE%D0%B3%D0%BE%D0%B2%D1%8B%D0%B5-%D0%BA%D0%BE%D0%BD%D1%84%D0%B8%D0%B3%D1%83%D1%80%D0%B0%D1%86%D0%B8%D0%B8-%D1%83%D1%81%D1%82%D1%80%D0%BE%D0%B9%D1%81%D1%82%D0%B2)

**Цели:**
- Настроить OSPFv2 для Underlay в сети CLOS;

 **Реализовать схему OSPF on UNDERLAY** по схеме:
![image](https://user-images.githubusercontent.com/60564360/212604737-1ebb5ba8-e693-4d69-abcd-7f6462495156.png)


##  1. <a name=''></a>Введение



Настроим устройства для дальнейшего построения лабораторных работ :

##  4. <a name='OSPFv2CiscoNexusv9500.'></a>1. Базовые настройки OSPFv2 для Cisco Nexus v9500.


Базовые настройки OSPFv2 для Cisco Nexus v9500: 

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


Включим протокол OSPFv2 выполним базовую настройку в одной зоне
```
feature ospf
router ospf UNDERLAY
 router-id 10.8.1.0
 log-adjacency-changes
 passive-interface default

key chain OSPF
  key 0
    key-string 7 070e771a190d4d521611085d5c2f2a722a

```
* `router ospf UNDERLAY` - указание строгового параметра в качетве ID OSPFv2 процесса
* `router-id` - укажем в качетве RID адресс lo1 на каждом устройстве
* `log-adjacency-changes` - используется для отлеживания с логах состояние OSPF соседства
* `passive interface default` - по умолчанию всех интерфейсы не учавствуют в OSPF
* `key chain OSPF` - настройка MD5 ключа который  может использоватся для авторизации в OSPFv2


**Донастроим P2P интерфейсы на всех свитчах:**

```
conf t
interface Ethernet1/1-3
  ip router ospf UNDERLAY area 0.0.0.0
  ip ospf network point-to-point
  mtu 9192
  no ip ospf passive-interface
  ip ospf authentication message-digest
  ip ospf authentication key-chain OSPF
```

**Добавим lo1 интерфейсы на всех свитчах в OSPF:**
```
conf t
interface loopback1
  ip router ospf UNDERLAY area 0.0.0.0
```

* `ip router ospf UNDERLAY area 0.0.0.0` - укажем одинаковую backbone area для нужных интефейсах
* `point-to-point` - в сетях CLOS'а обычно используется ip адресация линков с сетями `/31` между двумя устройствами, поэтому ethernet интерфейсы помечаются как как point-to-point (p2p) линки при включении их в OSPF, что позволяет избежать лишних проблем вызванные выбором DR/BDR в больших brodcast сетях.
* `mtu` - Для свитчей серии Cisco Nexus 9000, размер MTU может быть установлен до 9216.
Размер MTU должен быть настроен идентично на двух сторонах линка.
>Комманда `ip ospf mtu-ignore` - может использоватся для устанавления OSPF соседства при различающемся MTU, но возможна ситуация когда LSA пакеты больше минимального MTU не будут достигать соседнего устройства и статус соединения будет постоянно EXSTART.
* `no ip ospf passive-interface` - указание использование интерфейса в OSPFv2 протоколе
* `ip ospf authentication message-digest` - использование аутенфикации
* `ip ospf authentication key-chain OSPF` - указание ключа аутефикации (одинаковый на сех устройствах)



После того как все устройства будут настроены можно проверить статус протокола

Например на Spine01: 

```
Spine01# show ip ospf neighbors
 OSPF Process ID UNDERLAY VRF default
 Total number of neighbors: 2
 Neighbor ID     Pri State            Up Time  Address         Interface
 10.8.1.0          1 FULL/ -          1d16h    10.10.1.0       Eth1/6
 10.8.2.0          1 FULL/ -          00:04:56 10.10.2.0       Eth1/7

Leaf01# show ip ospf interface brief
 OSPF Process ID UNDERLAY VRF default
 Total number of interface: 3
 Interface               ID     Area            Cost   State    Neighbors Status
 Lo1                     1      0.0.0.0         1      LOOPBACK 0         up
 Eth1/6                  2      0.0.0.0         40     P2P      1         up
 Eth1/7                  3      0.0.0.0         40     P2P      0         up

```


И выполнить запросы между устройствами. например между свитчем **Spine01** и **Spine02**
```
Spine01# ping 10.8.2.0 source 10.8.1.0
PING 10.8.2.0 (10.8.2.0) from 10.8.1.0: 56 data bytes
Request 0 timed out
64 bytes from 10.8.2.0: icmp_seq=1 ttl=253 time=9.397 ms
64 bytes from 10.8.2.0: icmp_seq=2 ttl=253 time=11.731 ms
64 bytes from 10.8.2.0: icmp_seq=3 ttl=253 time=10.109 ms
64 bytes from 10.8.2.0: icmp_seq=4 ttl=253 time=7.556 ms

--- 10.8.2.0 ping statistics ---
5 packets transmitted, 4 packets received, 20.00% packet loss
round-trip min/avg/max = 7.556/9.698/11.731 ms

Spine01# traceroute 10.8.2.0 source 10.8.1.0
traceroute to 10.8.2.0 (10.8.2.0) from 10.8.1.0 (10.8.1.0), 30 hops max, 40 byte packets
 1  10.10.1.1 (10.10.1.1)  5.106 ms  3.238 ms  2.878 ms
 2  10.8.2.0 (10.8.2.0)  5.102 ms  5.684 ms  5.498 ms
```

Проверим таблицу машрутизации видно что доступны несколько путей с равными параметрами для задействования ECMP
```
Spine01# show ip route 10.8.2.0
IP Route Table for VRF "default"
'*' denotes best ucast next-hop
'**' denotes best mcast next-hop
'[x/y]' denotes [preference/metric]
'%<string>' in via output denotes VRF <string>

10.8.2.0/32, ubest/mbest: 2/0
    *via 10.10.1.1, Eth1/1, [110/81], 07:56:40, ospf-UNDERLAY, intra
    *via 10.10.1.5, Eth1/3, [110/81], 08:08:27, ospf-UNDERLAY, intra
```


##  8. <a name=':'></a>Итоговые конфигурации устройств:


`show run | section interface|router|key|feature|hostname`

**Spine01**

```
hostname Spine01
feature ospf

key chain OSPF
  key 0
    key-string 7 070e771a190d4d521611085d5c2f2a722a

interface Ethernet1/1
  description to-leaf-01
  mtu 9192
  ip address 10.10.1.0/31
  ip ospf authentication message-digest
  ip ospf authentication key-chain OSPF
  ip ospf network point-to-point
  no ip ospf passive-interface
  ip router ospf UNDERLAY area 0.0.0.0
  no shutdown

interface Ethernet1/2
  description to-leaf-02
  mtu 9192
  ip address 10.10.1.2/31
  ip ospf authentication message-digest
  ip ospf authentication key-chain OSPF
  ip ospf network point-to-point
  no ip ospf passive-interface
  ip router ospf UNDERLAY area 0.0.0.0
  no shutdown

interface Ethernet1/3
  description to-leaf-02
  mtu 9192
  ip address 10.10.1.4/31
  ip ospf authentication message-digest
  ip ospf authentication key-chain OSPF
  ip ospf network point-to-point
  no ip ospf passive-interface
  ip router ospf UNDERLAY area 0.0.0.0
  no shutdown

interface loopback1
  description for-underlay
  ip address 10.8.1.0/32
  ip router ospf UNDERLAY area 0.0.0.0

router ospf UNDERLAY
  router-id 10.8.1.0
  log-adjacency-changes
  passive-interface default

```

**Spine02**

```
hostname Spine02

feature ospf

key chain OSPF
  key 0
    key-string 7 070e771a190d4d521611085d5c2f2a722a

interface Ethernet1/1
  description to-leaf-01
  mtu 9192
  ip address 10.10.2.0/31
  ip ospf authentication message-digest
  ip ospf authentication key-chain OSPF
  ip ospf network point-to-point
  no ip ospf passive-interface
  ip router ospf UNDERLAY area 0.0.0.0
  no shutdown

interface Ethernet1/2
  description to-leaf-02
  mtu 9192
  ip address 10.10.2.2/31
  ip ospf authentication message-digest
  ip ospf authentication key-chain OSPF
  ip ospf network point-to-point
  no ip ospf passive-interface
  ip router ospf UNDERLAY area 0.0.0.0
  no shutdown

interface Ethernet1/3
  description to-leaf-03
  mtu 9192
  ip address 10.10.2.4/31
  ip ospf authentication message-digest
  ip ospf authentication key-chain OSPF
  ip ospf network point-to-point
  no ip ospf passive-interface
  ip router ospf UNDERLAY area 0.0.0.0
  no shutdown

interface loopback1
  ip address 10.8.2.0/32
  ip router ospf UNDERLAY area 0.0.0.0

router ospf UNDERLAY
  router-id 10.8.2.0
  log-adjacency-changes
  passive-interface default


```
***
**Leaf01**

```
hostname Leaf01

feature ospf

key chain OSPF
  key 0
    key-string 7 070e771a190d4d521611085d5c2f2a722a
interface Ethernet1/1
  description to-PC1
  ip address 10.12.1.254/24
  no shutdown

interface Ethernet1/6
  description to-spine-01
  mtu 9192
  ip address 10.10.1.1/31
  ip ospf authentication message-digest
  ip ospf authentication key-chain OSPF
  ip ospf network point-to-point
  no ip ospf passive-interface
  ip router ospf UNDERLAY area 0.0.0.0
  no shutdown

interface Ethernet1/7
  description to-spine-02
  mtu 9192
  ip address 10.10.2.1/31
  ip ospf authentication message-digest
  ip ospf authentication key-chain OSPF
  ip ospf network point-to-point
  no ip ospf passive-interface
  ip router ospf UNDERLAY area 0.0.0.0
  no shutdown

interface loopback1
  description for-underlay
  ip address 10.8.0.1/32
  ip router ospf UNDERLAY area 0.0.0.0

router ospf UNDERLAY
  router-id 10.8.0.1
  log-adjacency-changes
  passive-interface default
```

**Leaf02**

```
hostname Leaf02

feature ospf

key chain OSPF
  key 0
    key-string 7 070e771a190d4d521611085d5c2f2a722a

interface Ethernet1/1
  description to-PC2
  ip address 10.12.2.254/24
  no shutdown

interface Ethernet1/6
  description to-spine-01
  mtu 9192
  ip address 10.10.1.3/31
  ip ospf authentication message-digest
  ip ospf authentication key-chain OSPF
  ip ospf network point-to-point
  no ip ospf passive-interface
  ip router ospf UNDERLAY area 0.0.0.0
  no shutdown

interface Ethernet1/7
  description to-spine-02
  mtu 9192
  ip address 10.10.2.3/31
  ip ospf authentication message-digest
  ip ospf authentication key-chain OSPF
  ip ospf network point-to-point
  no ip ospf passive-interface
  ip router ospf UNDERLAY area 0.0.0.0
  no shutdown

interface loopback1
  ip address 10.8.0.2/32
  ip router ospf UNDERLAY area 0.0.0.0

router ospf UNDERLAY
  router-id 10.8.1.0
  log-adjacency-changes
  passive-interface default
```

**Leaf03**
```
hostname Leaf03

feature ospf

key chain OSPF
  key 0
    key-string 7 070e771a190d4d521611085d5c2f2a722a

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
  mtu 9192
  ip address 10.10.1.5/31
  ip ospf authentication message-digest
  ip ospf authentication key-chain OSPF
  ip ospf network point-to-point
  no ip ospf passive-interface
  ip router ospf UNDERLAY area 0.0.0.0
  no shutdown

interface Ethernet1/7
  description to-spine-02
  mtu 9192
  ip address 10.10.2.5/31
  ip ospf authentication message-digest
  ip ospf authentication key-chain OSPF
  ip ospf network point-to-point
  no ip ospf passive-interface
  ip router ospf UNDERLAY area 0.0.0.0
  no shutdown

interface loopback1
  description for-UNDERLAY
  ip address 10.8.0.3/32
  ip router ospf UNDERLAY area 0.0.0.0

router ospf UNDERLAY
  router-id 10.8.0.3
  log-adjacency-changes
  passive-interface default
```

