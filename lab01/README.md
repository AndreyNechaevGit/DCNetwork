# Первая лабораторная работа

[Таблица IP адресов](https://github.com/AndreyNechaevGit/DCNetwork/blob/main/lab01/README.md#%D1%82%D0%B0%D0%B1%D0%BB%D0%B8%D1%86%D0%B0-ip-%D0%B0%D0%B4%D1%80%D0%B5%D1%81%D0%BE%D0%B2)

[Итоговая конфигурация устройств:](https://github.com/AndreyNechaevGit/DCNetwork/tree/main/lab01#%D0%B8%D1%82%D0%BE%D0%B3%D0%BE%D0%B2%D1%8B%D0%B5-%D0%BA%D0%BE%D0%BD%D1%84%D0%B8%D0%B3%D1%83%D1%80%D0%B0%D1%86%D0%B8%D0%B8-%D1%83%D1%81%D1%82%D1%80%D0%BE%D0%B9%D1%81%D1%82%D0%B2)

## Проектирование адресного пространства в CLOS сети

**Цели:**

- Собрать схему CLOS;
- Распределить адресное пространство для Underlay сети;

### Введение

- **Spine** - используется связи **Leaf** устройств между собой <br>
- **Leaf** - используется для подключения конечных **устройств** хостов, Firewall, Borders и т.д.<br>
- **POD** (Point Of Delivery) — обособленная группа устройств, где _Spine_ первого уровня подключается к _Spine_ второго уровня, т.е. некий "кирпичик" ЦОДа.<br>
- **loopback 01** - для динамической маршрутизации Underlay<br>
- **loopback 02** - для динамической маршрутизации Overlay<br>

**Реализовать схему топологию CLOS** по схеме:
![image](https://user-images.githubusercontent.com/60564360/209561759-29e8faa6-6589-4280-a89a-70402c4a5fad.png)

**Схема IP адресации устройств**

10.Dn.Sn.x/13 <br>

- Dn - DC Number (for DC2 (8-15))
- Sn - Spine Number
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
10.8.Y.0/32 для **lo0** Spine Y<br>
10.9.Y.0/32 для **lo1** Spine Y<br>
10.8.0.X/32 для **lo0** Leaf X<br>
10.9.0.X/32 для **lo1** Leaf X<br>

> Например: <br>
> Spine2 lo1 10.8.2.0/32<br>
> Leaf1 lo2 10.9.0.1/32 <br>
> Spine2-Leaf1 p2p 10.9.2.0/31 <br>

#### Таблица IP адресов

| Device | Interface |     IP Address |
| ------ | --------- | -------------: |
| Spine1 | e1/1      |   10.10.1.0/31 |
| -      | e1/2      |   10.10.1.2/31 |
| -      | e1/3      |   10.10.1.4/31 |
| -      | lo1       |    10.8.1.0/32 |
| -      | lo2       |    10.9.1.0/32 |
| Spine2 | e1/1      |   10.10.2.0/31 |
| -      | e1/2      |   10.10.2.2/31 |
| -      | e1/3      |   10.10.2.4/31 |
| -      | lo1       |    10.8.2.0/32 |
| -      | lo2       |    10.9.2.0/32 |
| Leaf1  | e1/1      | 10.12.1.254/24 |
| -      | e1/6      |   10.10.1.1/31 |
| -      | e1/7      |   10.10.2.1/31 |
| -      | lo1       |    10.8.0.1/32 |
| -      | lo2       |    10.9.0.1/32 |
| Leaf2  | e1/1      | 10.12.2.254/24 |
| -      | e1/6      |   10.10.1.3/31 |
| -      | e1/7      |   10.10.2.3/31 |
| -      | lo1       |    10.8.0.2/32 |
| -      | lo2       |    10.9.0.2/32 |
| Leaf3  | e1/1      | 10.12.3.254/24 |
| -      | e1/6      |   10.10.1.5/31 |
| -      | e1/7      |   10.10.2.5/31 |
| -      | lo1       |    10.8.0.3/32 |
| -      | lo2       |    10.9.0.3/32 |
| PC1    | NIC       |   10.12.1.1/24 |
| PC2    | NIC       |   10.12.2.1/24 |
| PC3    | NIC       |   10.12.3.1/24 |
| PC4    | NIC       |   10.12.3.2/24 |

<details>
<summary> Старые значения для looback: </summary>

| Device | Interface |  IP Address |
| ------ | --------- | ----------: |
| Spine1 | lo1       | 10.8.1.0/32 |
| -      | lo2       | 10.9.1.0/32 |
| Spine2 | lo1       | 10.8.2.1/32 |
| -      | lo2       | 10.9.2.1/32 |
| Leaf1  | lo1       | 10.8.1.3/32 |
| -      | lo2       | 10.9.1.3/32 |
| Leaf2  | lo1       | 10.8.1.4/32 |
| -      | lo2       | 10.9.1.4/32 |
| Leaf3  | lo1       | 10.8.1.5/32 |
| -      | lo2       | 10.9.1.5/32 |

</details>

<br>

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
Spine01(config-if)# do sh running-config interface e1/1
!Command: show running-config interface Ethernet1/1
!Running configuration last done at: Fri Dec 16 17:50:00 2022
!Time: Mon Dec 19 05:46:28 2022
version 9.3(10) Bios:version
interface Ethernet1/1
  description to-leaf-01
  ip address 10.10.1.0/31
  no shutdown

Spine01(config)# sh ip int brief

IP Interface Status for VRF "default"(1)
Interface            IP Address      Interface Status
Lo1                  10.8.1.0        protocol-up/link-up/admin-up
Lo2                  10.9.1.0        protocol-up/link-up/admin-up
Eth1/1               10.10.1.0       protocol-up/link-up/admin-up
Eth1/2               10.10.1.2       protocol-up/link-up/admin-up
Eth1/3               10.10.1.4       protocol-up/link-up/admin-up

Spine01(config)# sh cdp neighbors
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
PC1> ip 10.12.1.1 255.255.255.0 10.12.1.254
Checking for duplicate address...
PC1 : 10.12.1.1 255.255.255.0 gateway 10.12.1.254
PC1> save
Saving startup configuration to startup.vpc
.  done
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
