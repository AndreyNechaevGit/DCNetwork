# Вторая лабораторная работа

[Таблица IP адресов](https://github.com/AndreyNechaevGit/DCNetwork/blob/main/lab01/README.md#%D1%82%D0%B0%D0%B1%D0%BB%D0%B8%D1%86%D0%B0-ip-%D0%B0%D0%B4%D1%80%D0%B5%D1%81%D0%BE%D0%B2)

[Итоговая конфигурация устройств:](https://github.com/AndreyNechaevGit/DCNetwork/tree/main/lab01#%D0%B8%D1%82%D0%BE%D0%B3%D0%BE%D0%B2%D1%8B%D0%B5-%D0%BA%D0%BE%D0%BD%D1%84%D0%B8%D0%B3%D1%83%D1%80%D0%B0%D1%86%D0%B8%D0%B8-%D1%83%D1%81%D1%82%D1%80%D0%BE%D0%B9%D1%81%D1%82%D0%B2)

## Построение Underlay сети(OSPF)

**Цели:**

- Настроить OSPF для Underlay сети;


### Введение
OSPF
Designated Router (DR) и Backup Designated Router (BDR).

- **BFD** - используется связи **Leaf** устройств между собой <br>
- **ECMP** - используется для подключения конечных **устройств** хостов, Firewall, Borders и т.д.<br>
- **Area 0** (Point Of Delivery) — обособленная группа устройств, где _Spine_ первого уровня подключается к _Spine_ второго уровня, т.е. некий "кирпичик" ЦОДа.<br>
- loopback 01 - для динамической маршрутизации Underlay <br>
- loopback 02 - для динамической маршрутизации Overlay <br>

BFD is not supported for OSPFv3!!
Network types are as follows:

Point-to-point—A network that exists only between two routers. All neighbors on a point-to-point network establish adjacency and there is no DR.
Broadcast—A network with multiple routers that can communicate over a shared medium that allows broadcast traffic, such as Ethernet. OSPFv2 routers establish a DR and BDR that controls LSA flooding on the network. OSPFv2 uses the well-known IPv4 multicast addresses 224.0.0.5 and a MAC address of 0100.5300.0005 to communicate with neighbors.

Table 6-2 Default OSPFv2 Parameters

Parameters
Default
Hello interval

10 seconds

Dead interval

40 seconds

Graceful restart grace period

60 seconds

OSPFv2 feature

Disabled

Stub router advertisement announce time

600 seconds

Reference bandwidth for link cost calculation

40 Gb/s

LSA minimal arrival time

1000 milliseconds

LSA group pacing

10 seconds

SPF calculation initial delay time

200 milliseconds

SPF minimum hold time

5000 milliseconds

SPF calculation initial delay time

1000 milliseconds



**Реализовать схему топологию CLOS** по схеме:
![image](https://user-images.githubusercontent.com/60564360/209561759-29e8faa6-6589-4280-a89a-70402c4a5fad.png)



Настроим устройства для дальнейшего построения лабораторных работ, :

1. Базовые настройки для Cisco Nexus v9500.
2. Назначим IP адреса на все интерфейсы согласно Таблица адресов.

#### Настройка корректной загрузки Cisco Nexus v9500 v9.3.10

<details>
<summary> На всех добавленных образах выполнить при начальной загрузке: </summary>

```
loader > dir
loader > boot nxos.9.у.х.bin
```

**затем добавить в конфиг путь к загрузочному образу:**

```
switch # conf t
Enter configuration commands, one per line. End with CNTL/Z.
Leaf01(config)# boot nxos bootflash:/nxos.9.3.10.bin sup-1
Performing image verification and compatibility check, please wait....
Leaf01(config)# do copy running-config startup-config
[########################################] 100%
Copy complete, now saving to disk (please wait)...
Copy complete.
```

</details>

#### На всех устройствах назначим корректный hostname

```
switch# hostname Spine01
```

### Назначение IP адресов на стенде:

#### Пример настройки Spine/Leaf

```
Spine01(config)# interface ethernet 1/1
Spine01(config-if)# ip address 10.9.1.0 255.255.255.252
Spine01(config-if)# exit
Spine01(config)# interface lo1
Spine01(config-if)# ip address 10.8.1.0 255.255.255.255
```

<details>
<summary> Проверка параметров: </summary>

```
 
 sh ip ospf neighbor
 clear ip ospf process.
 
Leaf-Switch-V1# show ip ospf

 Routing Process UNDERLAY with ID 10.1.1.54 VRF default
 Routing Process Instance Number 1
 Stateful High Availability enabled
 Graceful-restart is configured
   Grace period: 60 state: Inactive 
   Last graceful restart exit status: None
 Supports only single TOS(TOS0) routes
 Supports opaque LSA
 Administrative distance 110
 Reference Bandwidth is 40000 Mbps
 SPF throttling delay time of 200.000 msecs,
   SPF throttling hold time of 1000.000 msecs, 
   SPF throttling maximum wait time of 5000.000 msecs
 LSA throttling start time of 0.000 msecs,
   LSA throttling hold interval of 5000.000 msecs, 
   LSA throttling maximum wait time of 5000.000 msecs
 Minimum LSA arrival 1000.000 msec
 LSA group pacing timer 10 secs
 Maximum paths to destination 8
 Number of external LSAs 0, checksum sum 0
 Number of opaque AS LSAs 0, checksum sum 0
 Number of areas is 1, 1 normal, 0 stub, 0 nssa
 Number of active areas is 1, 1 normal, 0 stub, 0 nssa
 Install discard route for summarized external routes.
 Install discard route for summarized internal routes.
   Area BACKBONE(0.0.0.0) 
        Area has existed for 03:12:54
        Interfaces in this area: 2 Active interfaces: 2
        Passive interfaces: 0  Loopback interfaces: 1
        No authentication available
        SPF calculation has run 5 times
         Last SPF ran for 0.000195s
        Area ranges are
        Number of LSAs: 3, checksum sum 0x196c2

Leaf-Switch-V1# show ip ospf interface

loopback0 is up, line protocol is up
    IP address 10.1.1.54/32
    Process ID UNDERLAY VRF default, area 0.0.0.0
    Enabled by interface configuration
    State LOOPBACK, Network type LOOPBACK, cost 1
    Index 1
 Ethernet1/41 is up, line protocol is up
    Unnumbered interface using IP address of loopback0 (10.1.1.54)
    Process ID UNDERLAY VRF default, area 0.0.0.0
    Enabled by interface configuration
    State P2P, Network type P2P, cost 4
    Index 2, Transmit delay 1 sec
    1 Neighbors, flooding to 1, adjacent with 1
    Timer intervals: Hello 10, Dead 40, Wait 40, Retransmit 5
      Hello timer due in 00:00:07
    No authentication
    Number of opaque link LSAs: 0, checksum sum 0

Leaf-Switch-V1# show ip ospf neighbors 

OSPF Process ID UNDERLAY VRF default
 Total number of neighbors: 1
 Neighbor ID     Pri State            Up Time  Address         Interface
 10.1.1.53          1 FULL/ -          06:18:32 10.1.1.53        Eth1/41


```

</details>

#### Пример настройки PC

```
VPCS> set pcname PC1
PC1> ip 10.12.0.1 255.255.255.0 10.12.0.254
Checking for duplicate address...
PC1 : 10.12.0.1 255.255.255.0 gateway 10.12.0.254
```

<details>
<summary> Проверка: </summary>

```
PC1> ping  10.12.0.254

10.12.0.254 icmp_seq=1 timeout
84 bytes from 10.12.0.254 icmp_seq=2 ttl=255 time=9.732 ms
84 bytes from 10.12.0.254 icmp_seq=3 ttl=255 time=7.680 ms
84 bytes from 10.12.0.254 icmp_seq=4 ttl=255 time=7.632 ms
84 bytes from 10.12.0.254 icmp_seq=5 ttl=255 time=7.306 ms

```

Проверка параметров:

```
PC1> show ip

NAME        : PC1[1]
IP/MASK     : 10.12.0.1/24
GATEWAY     : 10.12.0.254
DNS         :
MAC         : 00:50:79:66:68:0a
LPORT       : 20000
RHOST:PORT  : 127.0.0.1:30000
MTU         : 1500
```

</details>

### Итоговые конфигурации устройств:

**Spine01**

```
interface Ethernet1/1
  description to-leaf-01
  ip address 10.10.1.0/31
  no shutdown

interface Ethernet1/2
  description to-leaf-02
  ip address 10.10.1.2/31
  no shutdown
  
interface Ethernet1/3
  description to-leaf-02
  ip address 10.10.1.4/31
  no shutdown
  
interface loopback1
  ip address 10.8.1.0/32

interface loopback2
  ip address 10.9.1.0/32

```

**Spine02**

```
interface Ethernet1/1
  description to-leaf-01
  ip address 10.10.2.0/31
  no shutdown

interface Ethernet1/2
  description to-leaf-02
  ip address 10.10.2.2/31
  no shutdown
  
interface Ethernet1/3
  description to-leaf-03
  ip address 10.10.2.4/31
  no shutdown
  
interface loopback1
  ip address 10.8.2.0/32

interface loopback2
  ip address 10.9.2.0/32

```

**Leaf01**

```
interface Ethernet1/1
  description to-PC1
  ip address 10.12.1.254/24
  no shutdown

interface Ethernet1/6
  description to-spine-01
  ip address 10.10.1.1/31
  no shutdown
  
interface Ethernet1/7
  description to-spine-02
  ip address 10.10.2.1/31
  no shutdown
  
interface loopback1
  ip address 10.8.0.1/32

interface loopback2
  ip address 10.9.0.1/32

```

**Leaf02**

```
interface Ethernet1/1
  description to-PC2
  ip address 10.12.2.254/24
  no shutdown

interface Ethernet1/6
  description to-spine-01
  ip address 10.10.1.3/31
  no shutdown
  
interface Ethernet1/7
  description to-spine-02
  ip address 10.10.2.3/31
  no shutdown
  
interface loopback1
  ip address 10.8.0.2/32

interface loopback2
  ip address 10.9.0.2/32


```

**Leaf03**

```
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
  ip address 10.10.1.5/31
  no shutdown
  
interface Ethernet1/7
  description to-spine-02
  ip address 10.10.2.5/31
  no shutdown
  
interface loopback1
  ip address 10.8.0.3/32

interface loopback2
  ip address 10.9.0.3/32
  
```
