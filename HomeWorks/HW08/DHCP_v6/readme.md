# Лабораторная работа. Настройка DHCPv6.

## Топология:
![](Top.jpg)

## Таблица адресации:

|   Устройство   |   Интерфейс   |   IPv6-адрес             |
|  ------------  |  -----------  |  ---------------------   |
|  R1            |  G0/0/0       |  2001:db8:acad:2::1/64   |
|  R1            |  G0/0/0       |  fe80::1                 |
|  R1            |  G0/0/1       |  2001:db8:acad:1::1/64   |
|  R1            |  G0/0/1       |  fe80::1                 |
|  R2            |  G0/0/0       |  2001:db8:acad:2::2/64   |
|  R2            |  G0/0/0       |  fe80::2                 |
|  R2            |  G0/0/1       |  2001:db8:acad:3::1/64   |
|  R2            |  G0/0/1       |  fe80::1                 |
|  PC-A          |  NIC          |  DHCP                    |
|  PC-B          |  NIC          |  DHCP                    |

## Задачи:

### Часть 1. Создание сети и настройка основных параметров устройства.

### Часть 2. Проверка назначения адреса SLAAC от R1.

### Часть 3. Настройка и проверка сервера DHCPv6 без гражданства на R1.

### Часть 4. Настройка и проверка состояния DHCPv6 сервера на R1.

### Часть 5. Настройка и проверка DHCPv6 Relay на R2.

## Решение:

### Часть 1. Создание сети и настройка основных параметров устройства.

a.) Создайте сеть согласно топологии.

b.) Настройте базовые параметры каждого коммутатора.

```
Switch>en
Switch#conf t
Switch(config)#hostname S1
S1(config)#no ip domain-lookup 
S1(config)#enable secret class
S1(config)#line con 0
S1(config-line)#password cisco
S1(config-line)#line vty 0 4
S1(config-line)#password cisco
S1(config-line)#login
S1(config-line)#exit
S1(config)#service password-encryption 
S1(config)#banner motd #Authorized Access Only!#
S1(config)#int ra f0/1-4, f0/7-24, g0/1-2
S1(config-if-range)#sh
S1#copy run sta
Destination filename [startup-config]? 
Building configuration...
[OK]
```

```
Switch>en
Switch#conf t
Switch(config)#hostname S2
S2(config)#no ip domain-lookup 
S2(config)#enable secret class
S2(config)#line con 0
S2(config-line)#line con 0
S2(config-line)#password cisco
S2(config-line)#login
S2(config-line)#line vty 0 4
S2(config-line)#password cisco
S2(config-line)#login
S2(config-line)#exit
S2(config)#service password-encryption 
S2(config)#banner motd #Authorized Access Only!#	
S2(config)#int ra f0/1-4, f0/6-17, f0/19-24, g0/1-2
S2(config-if-range)#sh
S2#copy run sta
Destination filename [startup-config]? 
Building configuration...
[OK]
```

c.) Произведите базовую настройку маршрутизаторов.

```
Router>en
Router#conf t
Router(config)#hostname R1
R1(config)#no ip domain-lookup 
R1(config)#enable secret class
R1(config)#line con 0
R1(config-line)#password cisco
R1(config-line)#login
R1(config-line)#line vty 0 4
R1(config-line)#password cisco
R1(config-line)#login
R1(config-line)#exit
R1(config)#service pas
R1(config)#service password-encryption 
R1(config)#banner motd #Authorized Access Only!#
R1(config)#ipv6 unicast-routing 
R1#copy run sta
Destination filename [startup-config]? 
Building configuration...
[OK]
```

```
Router>en
Router#conf t
Router(config)#hostname R2
R2(config)#no ip domain-lookup 
R2(config)#enable secret class
R2(config)#line con 0
R2(config-line)#password cisco
R2(config-line)#login
R2(config-line)#line vty 0 4
R2(config-line)#password cisco
R2(config-line)#login
R2(config-line)#exit
R2(config-line)#exit
R2(config)#service password-encryption 
R2(config)#banner motd #Authorized Access Only!#
R2(config)#ipv6 unicast-routing 
R2#copy run sta
Destination filename [startup-config]? 
Building configuration...
[OK]
```

d.) Настройка интерфейсов и маршрутизации для обоих маршрутизаторов.

```
R1(config)#int g0/0/0
R1(config-if)#ipv6 address 2001:db8:acad:2::1/64
R1(config-if)#ipv6 address fe80::1 link-local
R1(config-if)#no sh
R1(config-if)#int g0/0/1
R1(config-if)#ipv6 address 2001:db8:acad:1::1/64
R1(config-if)#ipv6 address fe80::1 link-local 
R1(config-if)#no sh
R1(config)#ipv6 route ::/0 2001:db8:acad:2::2
R1#ping 2001:db8:acad:3::1

Type escape sequence to abort.
Sending 5, 100-byte ICMP Echos to 2001:db8:acad:3::1, timeout is 2 seconds:
!!!!!
Success rate is 100 percent (5/5), round-trip min/avg/max = 0/3/11 ms
R1#copy run sta
Destination filename [startup-config]? 
Building configuration...
[OK]
```

```
R2(config)#int g0/0/0
R2(config-if)#ipv6 address 2001:db8:acad:2::2/64
R2(config-if)#ipv6 address fe80::2 link-local 
R2(config-if)#no sh
R2(config-if)#int g0/0/1
R2(config-if)#ipv6 address 2001:db8:acad:3::1/64
R2(config-if)#ipv6 address fe80::1 link-local 
R2(config-if)#no sh
R2(config)#ipv6 route ::/0 2001:db8:acad:2::1
R2#copy run sta
Destination filename [startup-config]? 
Building configuration...
[OK]
```

### Часть 2. Проверка назначения адреса SLAAC от R1.




