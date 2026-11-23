# Лабораторная работа. Настройка и проверка расширенных списков контроля доступа.

## Топология:
![](Top.jpg)

## Таблица адресации:

|   Устройство   | interface    |   IP-адрес        |   Маска подсети  |  Шлюз по умолчанию |
|  ------------  |  ------------|  --------------   | -----------------| -------------------|
|  R1            |  G0/0/1      |  -                | -                | -                  |
|  R1            |  G0/0/1.20   |  10.20.0.1        | 255.255.255.0    | -                  |
|  R1            |  G0/0/1.30   |  10.30.0.1        | 255.255.255.0    | -                  |
|  R1            |  G0/0/1.40   |  10.40.0.1        | 255.255.255.0    | -                  |
|  R1            |  G0/0/1000   |  -                | -                | -                  |
|  R1            |  Loopback1   |  172.16.1.1       | 255.255.255.0    | -                  |
|  R2            |  G0/0/1      |  10.20.0.4        | 255.255.255.0    | -                  |
|  S1            |  VLAN 20     |  10.20.0.2        | 255.255.255.0    | 10.20.0.1          |
|  S2            |  VLAN 20     |  10.20.0.3        | 255.255.255.0    | 10.20.0.1          |
|  PC-A          |  NIC         |  10.30.0.10       | 255.255.255.0    | 10.30.0.1          |
|  PC-B          |  NIC         |  10.40.0.10       | 255.255.255.0    | 10.40.0.1          |

## Таблица VLAN:

|   VLAN         |       Имя    |   Назначенный интерфейс                |        
|  ------------  |  ----------- |  ------------------------------------- |
|  30            |  Operations  |  S1: F0/6                              |
|  40            |  Sales       |  S2: F0/18                             |
|  999           |  ParkingLot  |  S1: F0/2-4, F0/7-24, G0/1-2           |
|  999           |  ParkingLot  |  S2: F0/2-4, F0/6-17, F0/19-24, G0/1-2 |
|  1000          |  Native      |  -                                     |

## Цели:

### Часть 1. Создание сети и настройка основных параметров устройства.

### Часть 2. Настройка и проверка списков расширенного контроля доступа.

### Часть 3. Настройте транков (магистральных каналов).

### Часть 4. Настройка маршрутизации.

### Часть 5. Настройка удаленного доступа.

## Решение:

### Часть 1. Создание сети и настройка основных параметров устройства.

Шаг 1. Создайте сеть согласно топологии.

Шаг 2. Произведите базовую настройку маршрутизаторов.

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
R1(config)#service password-encryption 
R1(config)#banner motd #Authorized Access Only!#
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
R2(config)#service password-encryption 
R2(config)#banner motd #Authorized Access Only!#
R2#copy run sta
Destination filename [startup-config]? 
Building configuration...
[OK]
```

Шаг 3. Настройте базовые параметры каждого коммутатора.

```
Switch>en
Switch#conf t
Switch(config)#hostname S1
S1(config)#no ip domain-lookup 
S1(config)#enable secret class
S1(config)#line con 0
S1(config-line)#password cisco
S1(config-line)#login
S1(config-line)#line vty 0 4
S1(config-line)#password cisco
S1(config-line)#login
S1(config-line)#exit
S1(config)#service password-encryption 
S1(config)#banner motd #Authorized Access Only!#
S1(config)#exit
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
S2(config-line)#password cisco
S2(config-line)#login
S2(config-line)#line vty 0 4
S2(config-line)#password cisco
S2(config-line)#login
S2(config-line)#exit
S2(config)#service password-encryption 
S2(config)#banner motd #Authorized Access Only!#
S2(config)#exit
S2#copy run sta
Destination filename [startup-config]? 
Building configuration...
[OK]
```

### Часть 2. Настройка сетей VLAN на коммутаторах.

Шаг 1. Создайте сети VLAN на коммутаторах.

```
S1(config)#vlan 20
S1(config-vlan)#name Management
S1(config-vlan)#vlan 30
S1(config-vlan)#name Operations
S1(config-vlan)#vlan 40
S1(config-vlan)#name Sales
S1(config-vlan)#vlan 999
S1(config-vlan)#name ParkingLot
S1(config-vlan)#vlan 1000
S1(config-vlan)#name Native
S1(config-vlan)#exit
S1(config)#int vlan 20
S1(config-if)#ip address 10.20.0.2 255.255.255.0
S1(config-if)#exit
S1(config)#ip default-gateway 10.20.0.1
S1(config)#int ra f0/2-4, f0/7-24, g0/1-2
S1(config-if-range)#sw mo ac
S1(config-if-range)#sw ac vlan 999
```

```
S2(config)#vlan 20
S2(config-vlan)#name Management
S2(config-vlan)#vlan 30
S2(config-vlan)#name Operations
S2(config-vlan)#vlan 40
S2(config-vlan)#name Sales
S2(config-vlan)#vlan 999
S2(config-vlan)#name ParkingLot
S2(config-vlan)#vlan 1000
S2(config-vlan)#name Native
S2(config-vlan)#int vlan 20
S2(config-if)#ip address 10.20.0.3 255.255.255.0
S2(config-if)#exit
S2(config)#ip default-gateway 10.20.0.1
S2(config)#int ra f0/2-4, f0/6-17, f0/19-24, g0/1-2
S2(config-if-range)#sw mo ac
S2(config-if-range)#sw ac vlan 999
```

Шаг 2. Назначьте сети VLAN соответствующим интерфейсам коммутатора.

```
S1(config)#int f0/6
S1(config-if)#sw mo ac
S1(config-if)#sw ac vlan 30
S1#sh vlan br

VLAN Name                             Status    Ports
---- -------------------------------- --------- -------------------------------
1    default                          active    Fa0/1, Fa0/5
20   Management                       active    
30   Operations                       active    Fa0/6
40   Sales                            active    
999  ParkingLot                       active    Fa0/2, Fa0/3, Fa0/4, Fa0/7
                                                Fa0/8, Fa0/9, Fa0/10, Fa0/11
                                                Fa0/12, Fa0/13, Fa0/14, Fa0/15
                                                Fa0/16, Fa0/17, Fa0/18, Fa0/19
                                                Fa0/20, Fa0/21, Fa0/22, Fa0/23
                                                Fa0/24, Gig0/1, Gig0/2
1000 Native                           active    
1002 fddi-default                     active    
1003 token-ring-default               active    
1004 fddinet-default                  active    
1005 trnet-default                    active 
```

```
S2(config)#int f0/18
S2(config-if)#sw mo ac
S2(config-if)#sw ac vlan 40
S2(config-if)#end
S2#sh vlan br

VLAN Name                             Status    Ports
---- -------------------------------- --------- -------------------------------
1    default                          active    Fa0/1, Fa0/5
20   Management                       active    
30   Operations                       active    
40   Sales                            active    Fa0/18
999  ParkingLot                       active    Fa0/2, Fa0/3, Fa0/4, Fa0/6
                                                Fa0/7, Fa0/8, Fa0/9, Fa0/10
                                                Fa0/11, Fa0/12, Fa0/13, Fa0/14
                                                Fa0/15, Fa0/16, Fa0/17, Fa0/19
                                                Fa0/20, Fa0/21, Fa0/22, Fa0/23
                                                Fa0/24, Gig0/1, Gig0/2
1000 Native                           active    
1002 fddi-default                     active    
1003 token-ring-default               active    
1004 fddinet-default                  active    
1005 trnet-default                    active    
```

### Часть 3. Настройте транки (магистральные каналы).

Шаг 1. Вручную настройте магистральный интерфейс F0/1.

```
S1(config)#int f0/1
S1(config-if)#sw mo tr
S1(config-if)#sw trunk native vlan 1000
S1(config-if)#sw tr allowed vlan 20,30,40,1000
S1(config-if)#end
S1#sh int tru
Port        Mode         Encapsulation  Status        Native vlan
Fa0/1       on           802.1q         trunking      1000

Port        Vlans allowed on trunk
Fa0/1       20,30,40,1000

Port        Vlans allowed and active in management domain
Fa0/1       20,30,40,1000

Port        Vlans in spanning tree forwarding state and not pruned
Fa0/1       20,30,40,1000
```

```
S2(config)#int f0/1
S2(config-if)#sw mo tr
S2(config-if)#sw tr native vlan 1000
S2(config-if)#sw tr allowed vlan 20,30,40,1000
S2(config-if)#end
S2#sh int tru
Port        Mode         Encapsulation  Status        Native vlan
Fa0/1       on           802.1q         trunking      1000

Port        Vlans allowed on trunk
Fa0/1       20,30,40,1000

Port        Vlans allowed and active in management domain
Fa0/1       20,30,40,1000

Port        Vlans in spanning tree forwarding state and not pruned
Fa0/1       none
```

Шаг 2. Вручную настройте магистральный интерфейс F0/5 на коммутаторе S1.

```
S1(config)#int f0/5
S1(config-if)#sw mo tr
S1(config-if)#sw tr native vlan 1000
S1(config-if)#sw tr allowed vlan 20,30,40,1000
S1(config-if)#end
S1#sh int tru
Port        Mode         Encapsulation  Status        Native vlan
Fa0/1       on           802.1q         trunking      1000
Fa0/5       on           802.1q         trunking      1000

Port        Vlans allowed on trunk
Fa0/1       20,30,40,1000
Fa0/5       20,30,40,1000

Port        Vlans allowed and active in management domain
Fa0/1       20,30,40,1000
Fa0/5       20,30,40,1000

Port        Vlans in spanning tree forwarding state and not pruned
Fa0/1       20,30,40,1000
Fa0/5       20,30,40,1000
```

### Часть 4. Настройте маршрутизацию.


