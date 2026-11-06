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

