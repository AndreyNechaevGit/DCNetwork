# Первая лабораторная работа

[[toc]]

{:toc}

[Purpose](https://github.com/hillaryfraley/jobbriefings#purpose)

[Daily Briefing](https://github.com/AndreyNechaevGit/DCNetwork/tree/main/lab01)


Первая лабораторная работа
Проектирование адресного пространства в CLOS сети
Таблица IP адресов
Введение
Настройка корректной загрузки Cisco Nexus v9500 v9.3.10
На всех устройствах назначим корректный hostname
Назначение IP адресов на стенде:
Пример настройки Spline/Leaf
Пример настройки PC
Итоговые конфигурации устройств:

## Проектирование адресного пространства в CLOS сети

**Цели:**
 - Собрать схему CLOS;
 - Распределить адресное пространство для Underlay сетио;

**Реализовать схему топологию CLOS** по схеме:
![image](https://user-images.githubusercontent.com/60564360/208510561-f4c64890-dd9d-4507-8a0c-e6102ba272d3.png)

**Схема IP адресации устройств**

10.Dn.Sn.x/13 <br>
- Dn - DC Number (for DC2 (8-15))
- Sn - Spline Number
- X - IP by order

**Пример сетей для ЦОД2:** <br>
**lo1** 10.8.0.0/16 <br>
**lo2** 10.9.0.0/16 <br>
**p2p** 10.10.0.0/16 <br>
**reserved** 10.11.0.0/16 <br>
**services** 10.12.0.0/14 <br>

**На устройствах принимаем** <br>
p2p линки - /31<br>
lo - интерфейсы /32<br>

>Например: <br>
Spline1 lo1 10.8.1.0/32<br>
Leaf1 lo2 10.9.1.0/32 <br>
Spline2-Leaf1 p2p 10.9.2.0/31 <br>



#### Таблица IP адресов

|Device|Interface|  IP Address  |
|---|---| ------------ |
Spline1|e1/1|10.10.1.0/31
-|e1/2|10.10.1.2/31
-|e1/3|10.10.1.4/31
-|lo1|10.8.1.0/32
-|lo2|10.9.1.0/32
Spline2|e1/1|10.10.2.0/31
-|e1/2|10.10.2.2/31
-|e1/3|10.10.2.4/31
-|lo1|10.8.2.1/32
-|lo2|10.9.2.1/32
Leaf1|e1/1|10.12.1.254/24
-|e1/6|10.10.1.1/31
-|e1/7|10.10.2.1/31
-|lo1|10.8.1.3/32
-|lo2|10.9.1.3/32
Leaf2|e1/1|10.12.2.254/24
-|e1/6|10.10.1.3/31
-|e1/7|10.10.2.3/31
-|lo1|10.8.1.4/32
-|lo2|10.9.1.4/32
Leaf3|e1/1|10.12.3.254/24
-|e1/6|10.10.1.5/31
-|e1/7|10.10.2.5/31
-|lo1|10.8.1.5/32
-|lo2|10.9.1.5/32
PC1|NIC|10.12.1.1/24
PC2|NIC|10.12.2.1/24
PC3|NIC|10.12.3.1/24
PC4|NIC|10.12.3.2/24

### Введение

- **Spline** - используется  связи **Leaf** устройств между собой <br> 
- **Leaf** - используется для подключения конечных **устройств** хостов, Firewall, Borders и т.д.<br>
- **POD** (Point Of Delivery) — обособленная группа  устройств,  где *Spline* первого уровня подключается к *Spline* второго уровня, т.е. некий "кирпичик" ЦОДа.<br>
- loopback 01 - для динамической маршрутизации Underlay 
- loopback 02- для динамической маршрутизации Overlay 


Настроим устройства для дальнейшего построения лабораторных работ, :

1) Базовые настройки для Cisco Nexus v9500.
2) Назначим IP адреса на все интерфейсы согласно Таблица адресов.


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
switch# hostname Spline01
```

### Назначение IP адресов на стенде:

#### Пример настройки Spline/Leaf

```
Spline01(config)# interface ethernet 1/1
Spline01(config-if)# ip address 10.9.1.0 255.255.255.252
Spline01(config-if)# exit
Spline01(config)# interface lo1
Spline01(config-if)# ip address 10.8.1.0 255.255.255.255
```

<details>
<summary> Проверка параметров: </summary>

```
Spline01(config-if)# do sh running-config interface e1/1
!Command: show running-config interface Ethernet1/1
!Running configuration last done at: Fri Dec 16 17:50:00 2022
!Time: Mon Dec 19 05:46:28 2022
version 9.3(10) Bios:version
interface Ethernet1/1
  description to-leaf-01
  ip address 10.10.1.0/31
  no shutdown

Spline01(config)# sh ip int brief

IP Interface Status for VRF "default"(1)
Interface            IP Address      Interface Status
Lo1                  10.8.1.0        protocol-up/link-up/admin-up
Lo2                  10.9.1.0        protocol-up/link-up/admin-up
Eth1/1               10.10.1.0       protocol-up/link-up/admin-up
Eth1/2               10.10.1.2       protocol-up/link-up/admin-up
Eth1/3               10.10.1.4       protocol-up/link-up/admin-up

Spline01(config)# sh cdp neighbors
Capability Codes: R - Router, T - Trans-Bridge, B - Source-Route-Bridge
                  S - Switch, H - Host, I - IGMP, r - Repeater,
                  V - VoIP-Phone, D - Remotely-Managed-Device,
                  s - Supports-STP-Dispute

Device-ID          Local Intrfce  Hldtme Capability  Platform      Port ID
Leaf01(927JNTV9I7N)
                    Eth1/1         141    R S s     N9K-C9500v    Eth1/6
Leaf02(9LH5WO687NC)
                    Eth1/2         156    R S s     N9K-C9500v    Eth1/6
Leaf03(9L8LOMSYRQS)
                    Eth1/3         155    R S s     N9K-C9500v    Eth1/6

Total entries displayed: 3


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

**Spline01**
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
**Spline02**
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
  ip address 10.8.2.1/32

interface loopback2
  ip address 10.9.2.1/32

```
**Leaf01**
```
interface Ethernet1/1
  description to-PC1
  ip address 10.12.1.254/24
  no shutdown

interface Ethernet1/6
  description to-spline-01
  ip address 10.10.1.1/31
  no shutdown
  
interface Ethernet1/7
  description to-spline-02
  ip address 10.10.2.1/31
  no shutdown
  
interface loopback1
  ip address 10.8.1.3/32

interface loopback2
  ip address 10.9.1.3/32

```
**Leaf02**
```
interface Ethernet1/1
  description to-PC2
  ip address 10.12.2.254/24
  no shutdown

interface Ethernet1/6
  description to-spline-01
  ip address 10.10.1.3/31
  no shutdown
  
interface Ethernet1/7
  description to-spline-02
  ip address 10.10.2.3/31
  no shutdown
  
interface loopback1
  ip address 10.8.1.4/32

interface loopback2
  ip address 10.9.1.4/32

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
  description to-spline-01
  ip address 10.10.1.5/31
  no shutdown
  
interface Ethernet1/7
  description to-spline-02
  ip address 10.10.2.5/31
  no shutdown
  
interface loopback1
  ip address 10.8.1.5/32

interface loopback2
  ip address 10.9.1.5/32
```
