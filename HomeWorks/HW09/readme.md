# Лабораторная работа. Конфигурация безопасности коммутатора.

## Топология:
![](Top.jpg)

## 1. Таблица адресации:

|   Устройство   | interface/vlan |   IP-адрес        |   Маска подсети  |  
|  ------------  |  ------------  |  --------------   | -----------------|
|  R1            |  G0/0/1        |  192.168.10.1     | 255.255.255.0    |
|  R1            |  Loopback 0    |  10.10.1.1        | 255.255.255.0    |
|  S1            |  VLAN 10       |  192.168.10.201   | 255.255.255.0    |
|  S2            |  VLAN 10       |  192.168.10.202   | 255.255.255.0    |
|  PC-A          |  NIC           |  DHCP             | 255.255.255.0    |
|  PC-B          |  NIC           |  DHCP             | 255.255.255.0    |

## Цели:

### Часть 1. Настройка основного сетевого устройства.

### Часть 2. Настройка сетей VLAN.

### Часть 3. Настройки безопасности коммутатора.

## Решение:

### Часть 1. Настройка основного сетевого устройства.

Шаг 1. Создайте сеть.

Шаг 2. Настройте маршрутизатор R1.

```
Router>enable
Router#configure terminal
Enter configuration commands, one per line.  End with CNTL/Z.
Router(config)#hostname R1
R1(config)#no ip domain lookup
R1(config)#ip dhcp excluded-address 192.168.10.1 192.168.10.9
R1(config)#ip dhcp excluded-address 192.168.10.201 192.168.10.202
R1(config)#ip dhcp relay information trust-all
R1(config)#!
R1(config)#ip dhcp pool Students
R1(dhcp-config)# network 192.168.10.0 255.255.255.0
R1(dhcp-config)# default-router 192.168.10.1
R1(dhcp-config)# domain-name CCNA2.Lab-11.6.1
R1(dhcp-config)#!
R1(dhcp-config)#interface Loopback0
R1(config-if)# ip address 10.10.1.1 255.255.255.0
R1(config-if)#!
R1(config-if)#interface GigabitEthernet0/0/1
R1(config-if)# description Link to S1
R1(config-if)# ip address 192.168.10.1 255.255.255.0
R1(config-if)# no shutdown
R1(config-if)#!
R1(config-if)#line con 0
R1(config-line)# logging synchronous
R1(config-line)# exec-timeout 0 0
R1#sh ip int br
Interface              IP-Address      OK? Method Status                Protocol 
GigabitEthernet0/0/0   unassigned      YES unset  administratively down down 
GigabitEthernet0/0/1   192.168.10.1    YES manual up                    up 
Loopback0              10.10.1.1       YES manual up                    up 
Vlan1                  unassigned      YES unset  administratively down down
```

Шаг 3. Настройка и проверка основных параметров коммутатора.

```
Switch>en
Switch#conf t
Switch(config)#hostname S1
S1(config)#no ip domain-lookup 
S1(config)#int f0/6
S1(config-if)#description Link to PC
S1(config-if)#int f0/1
S1(config-if)#description Link to S2
S1(config-if)#int f0/5
S1(config-if)#description Link to R1
S1(config)#ip default-gateway 192.168.10.1
```

```
Switch>en
Switch#conf t
Switch(config)#hostname S2
S2(config)#no ip domain-lookup 
S2(config)#int f0/18
S2(config-if)#description Link to PC
S2(config-if)#int f0/1
S2(config-if)#description Link to S1
S2(config)#ip default-gateway 192.168.10.1
```

### Часть 2. Настройка сетей VLAN.

```
S1(config)#vlan 10
S1(config-vlan)#nam
S1(config-vlan)#name Management
S1(config-vlan)#int vlan 10
S1(config-if)#ip address 192.168.10.201 255.255.255.0
S1(config-if)#vlan 333
S1(config-vlan)#name Native
S1(config-vlan)#vlan 999
S1(config-vlan)#name ParkingLot
```

```
S2(config)#vlan 10
S2(config-vlan)#name Management
S2(config-vlan)#int vlan 10
S2(config-if)#ip address 192.168.10.202 255.255.255.0
S2(config-if)#vlan 333
S2(config-vlan)#name Native
S2(config-vlan)#vlan 999
S2(config-vlan)#name ParkingLot
```




