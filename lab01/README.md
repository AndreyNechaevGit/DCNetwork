# Первая лабораторная работа

[[toc]]

## Проектирование адресного пространства

**Цели:**
 - Собрать схему CLOS;
 - Распределить адресное пространство;

**Реализовать схему топологию CLOS** по схеме

![image](https://user-images.githubusercontent.com/60564360/205107265-9fc91b42-3b8c-4933-9b48-067daf071be3.png)

Распределить адресное пространство для Underlay сети



**Схема адресации**
10.Dn.Sn.x/31
Sn - Spline Number
X - IP by order
Dn - DC Number for DC2 2 (8-15)

DC2 10.[8-15].xx.yy/14
lo1 10.8.xx.yy/32
lo2 10.9.xx.yy/32
p2p 10.9.xx.yy/30
reserved 10.10.xx.yy/30
services 10.11.xx.yy/14

spline 10.8.xx.yy/14
leaf 10.8.xx.yy/14

p2p/ 30
lo 10.8./32

Spline1 lo1 10.8.1.0/32
Spline1 p2p
Leaf1 lo2 10.9.1.0/32

Spline2-Leaf1 p2p 10.9.2.0/31
Spline2-Leaf2 p2p 10.9.2.2/31
Spline2-Leaf3 p2p 10.9.2.4/31

Spline1-Leaf1 p2p 10.9.1.0/31
Spline1-Leaf2 p2p 10.9.1.2/31
Spline1-Leaf3 p2p 10.9.1.4/31


#### Таблица IP адресов

|Device|Interface|IP Address|Subnet Mask|Default Gateway
|---|---|---|---|---|
Spline1|e1/1|10.9.1.0/31|
-|e1/2|10.9.1.2/31|
-|e1/3|10.9.1.4/31|
-|lo1|10.8.1.0/32|
-|lo2|10.9.1.0/32|
Spline2|e1/1||10.9.2.0/31|
-|e1/2|10.9.1.2/31|
-|e1/3|10.9.1.4/31|
-|lo1|10.8.2.1/32|
-|lo2|10.9.2.1/32|
Leaf1|e1/1|192.168.3.1|255.255.255.0|N/A|
-|e1/6|192.168.4.1|255.255.255.0|N/A|
-|e1/7|N/A|N/A|N/A|
-|lo1|10.8.1.2/32|
-|lo2|10.9.1.2/32|
Leaf2|e1/1|192.168.3.1|255.255.255.0|N/A|
-|e1/6|192.168.4.1|255.255.255.0|N/A|
-|e1/7|N/A|N/A|N/A|
-|lo1|10.8.1.3/32|
-|lo2|10.9.1.3/32|
Leaf3|e1/1|192.168.3.1|255.255.255.0|N/A|
-|e1/6|192.168.4.1|255.255.255.0|N/A|
-|e1/7|N/A|N/A|N/A|
-|lo1|10.8.1.4/32|
-|lo2|10.9.1.4/32|
PC1|NIC|192.168.3.3|255.255.255.0|192.168.3.1
PC2|NIC|192.168.4.3|255.255.255.0|192.168.4.1
PC3|NIC|192.168.3.3|255.255.255.0|192.168.3.1
PC4|NIC|192.168.4.3|255.255.255.0|192.168.4.1

### Введение

- **Spline** - используется  связи **Leaf** устройств между собой <br> 
- **Leaf** - используется для подключения конечных **устройств** хостов, Firewall, Borders<br>
- **POD** (Point Of Delivery) — обособленная группа  устройств,  где *Spline* первого уровня подключается к *Spline* второго уровня, т.е. некий "кирпичик" ЦОДа.<br>

И наоборот, устройства, находящиеся в разных VLAN'ах, невидимы друг для друга на канальном уровне, даже если они подключены к одному коммутатору, и связь между этими устройствами возможна только на сетевом и более высоких уровнях.


Далее настроим устройства для дальнейшего построения лабораторных работ, :

1) Включим и настроим сохранение настроек для Cisco Nexus v9500.
2) Назначим IP адреса на все интерфейсы согласно Таблица адресов.
3) Настройки по умолчанию (шаг №0).

### Подготовка стенда

#### Настройка PC-A

```
VPCS> set pcname PC1
PC1> ip 192.168.3.3 255.255.255.0 192.168.3.1
Checking for duplicate address...
PC1 : 192.168.3.3 255.255.255.0 gateway 192.168.3.1
```

<details>
<summary> Проверка: </summary>

```
VPCS> ping 192.168.3.3

192.168.3.3 icmp_seq=1 ttl=64 time=0.001 ms
192.168.3.3 icmp_seq=2 ttl=64 time=0.001 ms
192.168.3.3 icmp_seq=3 ttl=64 time=0.001 ms
192.168.3.3 icmp_seq=4 ttl=64 time=0.001 ms
192.168.3.3 icmp_seq=5 ttl=64 time=0.001 ms
```
Проверка параметров:
```
VPCS> show ip

NAME        : VPCS[1]
IP/MASK     : 192.168.3.3/24
GATEWAY     : 192.168.3.1
DNS         :
MAC         : 00:50:79:66:68:04
LPORT       : 20000
RHOST:PORT  : 127.0.0.1:30000
MTU         : 1500
```
</details>

Default gateway пока недоступен:
```
VPCS> ping 192.168.3.1
host (192.168.3.1) not reachable
```



#### Настройка Leaf1

```
switch# hostname Spline01
Spline01(config)# interface ethernet 1/1
Spline01(config-if)# ip address 10.9.1.0 255.255.255.252
```

<details>
<summary> Проверка: </summary>

```
VPCS> ping 192.168.3.3

192.168.3.3 icmp_seq=1 ttl=64 time=0.001 ms
192.168.3.3 icmp_seq=2 ttl=64 time=0.001 ms
192.168.3.3 icmp_seq=3 ttl=64 time=0.001 ms
192.168.3.3 icmp_seq=4 ttl=64 time=0.001 ms
192.168.3.3 icmp_seq=5 ttl=64 time=0.001 ms
```
Проверка параметров:
```
Spline01(config-if)# do sh running-config interface e1/1
Spline01# sh ip int brief

IP Interface Status for VRF "default"(1)
Interface            IP Address      Interface Status
Lo1                  10.8.1.0        protocol-up/link-up/admin-up
Lo2                  10.9.1.0        protocol-up/link-up/admin-up
Eth1/1               10.9.1.1        protocol-down/link-down/admin-down
```
</details>

Default gateway пока недоступен:
```
VPCS> ping 192.168.3.1
host (192.168.3.1) not reachable
```

#### Настройка Spline1

```
switch# hostname Spline01
Spline01(config)# interface ethernet 1/1
Spline01(config-if)# ip address 10.9.1.0 255.255.255.252
```

<details>
<summary> Проверка: </summary>

```
VPCS> ping 192.168.3.3

192.168.3.3 icmp_seq=1 ttl=64 time=0.001 ms
192.168.3.3 icmp_seq=2 ttl=64 time=0.001 ms
192.168.3.3 icmp_seq=3 ttl=64 time=0.001 ms
192.168.3.3 icmp_seq=4 ttl=64 time=0.001 ms
192.168.3.3 icmp_seq=5 ttl=64 time=0.001 ms
```
Проверка параметров:
```
Spline01(config-if)# do sh running-config interface e1/1
Spline01# sh ip int brief

IP Interface Status for VRF "default"(1)
Interface            IP Address      Interface Status
Lo1                  10.8.1.0        protocol-up/link-up/admin-up
Lo2                  10.9.1.0        protocol-up/link-up/admin-up
Eth1/1               10.9.1.1        protocol-down/link-down/admin-down
```
</details>

Default gateway пока недоступен:
```
VPCS> ping 192.168.3.1
host (192.168.3.1) not reachable
```

### 1. Проверка "свежевоткнутых" коммутаторов

По умолчанию на коммутаторах S1 и S2 интерфейсы – подняты. 
Тоесть трафик пойдет через них сразу, без дополнительной настройки. <br>

![img_18.png](test_switches.png)

Давайте сделаем ping с PC-A на PC-B и посмотрим, что датаграммы идут.

PC-A:
```
VPCS> ping 192.168.4.3
host (192.168.3.1) not reachable
```

можно видеть, что на интерфейс PC-B `eth0`, через через S1 и S2, дошли ARP-бродкасты с просьбой вернуть `gateway`:<br>
![img_6.png](switches_wireshark.png)

Пакеты долетели без всяких IP-шников и настроек комммутаторов.

Заметим, что `not reachable` выдается в случае, если недоступен `default gateway`.<br>
Далее попробуем настроить на маршрутизаторе `default gateway`... каким-нибудь топорным способом.

