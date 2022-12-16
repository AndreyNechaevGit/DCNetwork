# Первая лабораторная работа

[[toc]]

## Проектирование адресного пространства в CLOS сети

**Цели:**
 - Собрать схему CLOS;
 - Распределить адресное пространство;

**Реализовать схему топологию CLOS** по схеме:

![image](https://user-images.githubusercontent.com/60564360/205107265-9fc91b42-3b8c-4933-9b48-067daf071be3.png)

Распределить адресное пространство для Underlay сети

**Схема адресации**

10.Dn.Sn.x/13 <br>
- Dn - DC Number (for DC2 (8-15))
- Sn - Spline Number
- X - IP by order

**Пример сетей для ЦОД2:**
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




Настроим устройства для дальнейшего построения лабораторных работ, :

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


