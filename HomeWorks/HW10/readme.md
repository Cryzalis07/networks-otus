# Лабораторная работа. Настройка протокола OSPFv2 для одной области.

## Топология:
![](Top.jpg)

## 1. Таблица адресации:

|   Устройство   | interface    |   IP-адрес        |   Маска подсети  |  
|  ------------  |  ------------|  --------------   | -----------------|
|  R1            |  G0/0/1      |  10.53.0.1        | 255.255.255.0    |
|  R1            |  Loopback1   |  172.16.1.1       | 255.255.255.0    |
|  R2            |  G0/0/1      |  10.53.0.2        | 255.255.255.0    |
|  R2            |  Loopback1   |  192.168.1.1      | 255.255.255.0    |

## Цели:

### Часть 1. Создание сети и настройка основных параметров устройства.

### Часть 2. Настройка и проверка базовой работы протокола  OSPFv2 для одной области.

### Часть 3. Оптимизация и проверка конфигурации OSPFv2 для одной области.

## Решение:

### Часть 1. Создание сети и настройка основных параметров устройства.

Шаг 1. Шаг 1. Создайте сеть согласно топологии.

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

### Часть 2. Настройка и проверка базовой работы протокола  OSPFv2 для одной области.

Шаг 1. Настройте адреса интерфейса и базового OSPFv2 на каждом маршрутизаторе.

a.	Настройте адреса интерфейсов на каждом маршрутизаторе, как показано в таблице адресации выше.

```
R1(config)#int g0/0/1
R1(config-if)#ip ad
R1(config-if)#ip address 10.53.0.1 255.255.255.0
R1(config-if)#no sh
R1(config-if)#exit
R1(config)#int loopback 1
R1(config-if)#ip address 172.16.1.1 255.255.255.0
```

```
R2(config)#int g0/0/1
R2(config-if)#ip address 10.53.0.2 255.255.255.0
R2(config-if)#no sh
R2(config-if)#exit
R2(config)#int loopback 1
R2(config-if)#ip address 192.168.1.1 255.255.255.0
```

b.	Перейдите в режим конфигурации маршрутизатора OSPF, используя идентификатор процесса 56.

c.	Настройте статический идентификатор маршрутизатора для каждого маршрутизатора (1.1.1.1 для R1, 2.2.2.2 для R2).

d.	Настройте инструкцию сети для сети между R1 и R2, поместив ее в область 0.

e.	Только на R2 добавьте конфигурацию, необходимую для объявления сети Loopback 1 в область OSPF 0.

```
R1(config)#router ospf 56
R1(config-router)#router-id 1.1.1.1
R1(config-router)#network 10.53.0.0 0.0.0.255 area 0
```

```
R2(config)#router ospf 56
R2(config-router)#router-id 2.2.2.2
R2(config-router)#network 10.53.0.0 0.0.0.255 area 0
R2(config-router)#network 192.168.1.0 0.0.0.255 area 0
```

f.	Убедитесь, что OSPFv2 работает между маршрутизаторами. Выполните команду, чтобы убедиться, что R1 и R2 сформировали смежность.

```
R1#sh ip ospf neighbor 


Neighbor ID     Pri   State           Dead Time   Address         Interface
2.2.2.2           1   FULL/DR         00:00:31    10.53.0.2       GigabitEthernet0/0/1
```

```
R2#sh ip ospf neighbor 


Neighbor ID     Pri   State           Dead Time   Address         Interface
1.1.1.1           1   FULL/BDR        00:00:35    10.53.0.1       GigabitEthernet0/0/1
```

Какой маршрутизатор является DR? Какой маршрутизатор является BDR? Каковы критерии отбора?

Мы видим, что маршрутизатор 1.1.1.1 является BDR, 2.2.2.2 DR. Т.к. приоритет у них одинаковый, DR становится маршрутизатор с большим значением ID.

g.	На R1 выполните команду show ip route ospf, чтобы убедиться, что сеть R2 Loopback1 присутствует в таблице маршрутизации. 

```
R1#show ip route ospf
     192.168.1.0/32 is subnetted, 1 subnets
O       192.168.1.1 [110/2] via 10.53.0.2, 00:02:21, GigabitEthernet0/0/1
```

h.	Запустите Ping до адреса интерфейса R2 Loopback 1 из R1.

```
R1#ping 192.168.1.1

Type escape sequence to abort.
Sending 5, 100-byte ICMP Echos to 192.168.1.1, timeout is 2 seconds:
!!!!!
Success rate is 100 percent (5/5), round-trip min/avg/max = 0/0/3 ms
```

### Часть 3. Оптимизация и проверка конфигурации OSPFv2 для одной области.

Шаг 1. Реализация различных оптимизаций на каждом маршрутизаторе.







