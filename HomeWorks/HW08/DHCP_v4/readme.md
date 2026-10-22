# Лабораторная работа. Реализация DHCPv4.

## Топология:
![](Top.jpg)

## 1. Таблица адресации:

|   Устройство   |   Интерфейс   |   IP-адрес        |   Маска подсети  |   Шлюз по умолчанию  |
|  ------------  |  -----------  |  --------------   | -----------------|   -------------------|
|  R1            |  G0/0/0       |  10.0.0.1         | 255.255.255.252  |   -------------------|
|  R1            |  G0/0/1       |  --------------   | -----------------|   -------------------|
|  R1            |  G0/0/1.100   |  192.168.1.1      | 255.255.255.192  |                      |
|  R1            |  G0/0/1.200   |  192.168.1.65     | 255.255.255.224  |                      |
|  R1            |  G0/0/1.1000  |  --------------   | -----------------|   -------------------|
|  R2            |  G0/0/0       |  10.0.0.2         | 255.255.255.252  |   -------------------|
|  R2            |  G0/0/1       |  192.168.1.97     | 255.255.255.240  |                      |
|  S1            |  VLAN200      |  192.168.1.66     | 255.255.255.224  | 192.168.1.65         |
|  S2            |  VLAN1        |                   |                  |                      |
|  PC-A          |  NIC          |  DHCP             | DHCP             | DHCP                 |
|  PC-B          |  NIC          |  DHCP             | DHCP             | DHCP                 |

## 2.	Таблица VLAN:

|   VLAN         |   Имя         | Назначенный интерфейс       |
|  ------------  |  -----------  |  ---------------------------|
|   1            |   Нет         | S2: F0/18                   |
|   100          |   Clients     | S1: F0/6                    |
|   200          |   Management  | S1: VLAN 200                |
|   999          |   Parking_Lot | S1: F0/1-4, F0/7-24, G0/1-2 |
|   1000         |   Native      | --                          |

## Цели:

### Часть 1. Создание сети и настройка основных параметров устройства.

### Часть 2. Настройка и проверка двух серверов DHCPv4 на R1

### Часть 3. Настройка и проверка DHCP-ретрансляции на R2

## Решение:

### Часть 1:	Создание сети и настройка основных параметров устройства.

Шаг 1.	Создание схемы адресации.

Шаг 2.	Создайте сеть согласно топологии.

Шаг 3.	Произвести базовую настройку маршрутизаторов.

```
Router>en
Router#conf t
Router(config)#
Router(config)#no ip domain-lookup
Router(config)#hostname R1
R1(config)#enable secret class
R1(config)#line con 0 
R1(config-line)#password cisco
R1(config-line)#login
R1(config-line)#line vty 0 4
R1(config-line)#password cisco
R1(config-line)#login
R1(config-line)#exit
R1(config)#service password-encryption
R1(config)#banner motd #Authorized Access Only!#
R1(config)#^Z
R1#copy running-config startup-config 
Destination filename [startup-config]? 
Building configuration...
[OK]
R1#clock set 21:33:00 22 oct 2026
```

```
Router>en
Router#conf t
Router(config)#
Router(config)#no ip domain-lookup
Router(config)#hostname R2
R2(config)#enable secret class
R2(config)#line con 0 
R2(config-line)#password cisco
R2(config-line)#login
R2(config-line)#line vty 0 4
R2(config-line)#password cisco
R2(config-line)#login
R2(config-line)#exit
R2(config)#service password-encryption
R2(config)#banner motd #Authorized Access Only!#
R2(config)#^Z
R2#copy running-config startup-config 
Destination filename [startup-config]? 
Building configuration...
[OK]
R2#clock set 21:35:00 22 oct 2026
```

Шаг 4.	Настройка маршрутизации между сетями VLAN на маршрутизаторе R1.

```
R1(config)#int g0/0/1
R1(config-if)#no sh

```
