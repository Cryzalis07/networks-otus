# Лабораторная работа. Внедрение маршрутизации между виртуальными локальными сетями.

## Топология:
![](Top.jpg)

## Таблица адресации:

|   Устройство   |   Интерфейс   |   IP-адрес        |   Маска подсети  |    Шлюз по умолчанию  |           
|  ------------  |  -----------  |  --------------   | -----------------|  -------------------- |
|   R1           |   G0/0/1.10   |   192.168.10.1    |  255.255.255.0   |  --                   |
|                |   G0/0/1.20   |   192.168.20.1    |  255.255.255.0   |  --                   |
|                |   G0/0/1.30   |   192.168.30.1    |  255.255.255.0   |  --                   |
|                |   G0/0/1.1000 |   --              |  --              |  --                   |
|   S1           |   VLAN 10     |   192.168.10.11   |  255.255.255.0   |  192.168.10.1         |
|   S2           |   VLAN 10     |   192.168.10.12   |  255.255.255.0   |  192.168.10.1         |
|   PC-A         |   NIC         |   192.168.20.3    |  255.255.255.0   |  192.168.20.1         |
|   PC-B         |   NIC         |   192.168.30.3    |  255.255.255.0   |  192.168.30.1         |

## Таблица VLAN:

|   VLAN         |   Имя         |   Назначенный интерфейс        |    
|  ------------  |  -----------  |  -------------------------     | 
|  10            |  Management   |  S1: VLAN 10, S2: VLAN 10      |
|  20            |  Sales        |  S1: F0/6                      |
|  30            |  Operations   |  S2: F0/18                     |
|  999           |  Parking_Lot  |  S1: F0/2-4, F0/7-24, G0/1-2   |
|  999           |  Parking_Lot  |  S2: F0/2-17, F0/19-24, G0/1-2 |
|  999           |  Parking_Lot  |  S2: F0/2-17, F0/19-24, G0/1-2 |
|  999           |  Native       |  ----------------------------  |

## Задачи:

### Часть 1. Создание сети и настройка основных параметров устройства.

### Часть 2. Создание сетей VLAN и назначение портов коммутатора.

### Часть 3. Настройка транка 802.1Q между коммутаторами.

### Часть 4. Настройка маршрутизации между сетями VLAN.

### Часть 5. Проверка, что маршрутизация между VLAN работает.

## Решение:

### Часть 1. Создание сети и настройка основных параметров устройства.

Шаг 1. Создадим сеть согласно топологии в CISCO Packe Tracert.

Шаг 2. Настроим базовые параметры для маршрутизатора.

```
Router>en
Router#conf t
Router(config)#hostname R1
R1(config)#no ip domain-lookup 
R1(config)#enable secret class
R1(config)#line con 0
R1(config-line)#password cisco
R1(config-line)#login
R1(config)#line vty 0 4
R1(config-line)#password cisco
R1(config-line)#login
R1(config-line)#exit
R1(config)#service password-encryption
R1(config)#banner motd #
Enter TEXT message.  End with the character '#'.
Unauthorized access is strictly prohibited.#
R1#copy run sta
Destination filename [startup-config]? 
Building configuration...
[OK]
R1#clock set 20:24:00 22 sep 2026
```

Шаг 3. Настроим базовые параметры каждого коммутатора.

```
Switch>en
Switch#conf t
Switch(config)#hostname S1
S1(config)#line con 0
S1(config-line)#password cisco
S1(config-line)#login
S1(config-line)#line vty 0 4
S1(config-line)#password cisco
S1(config-line)#login
S1(config-line)#exit
S1(config)#service password-encryption
S1(config)#banner motd #
Enter TEXT message.  End with the character '#'.
Unauthorized access is strictly prohibited.#
S1(config)#exit
S1#clock set 20:31:00 22 sep 2026
S1#copy run sta
Destination filename [startup-config]? 
Building configuration...
[OK]
```

```
Switch>en
Switch#conf t
Switch(config)#hostname S2
S2(config)#line con 0
S2(config-line)#password cisco
S2(config-line)#login
S2(config-line)#line vty 0 4
S2(config-line)#password cisco
S2(config-line)#login
S2(config-line)#exit
S2(config)#service password-encryption
S2(config)#banner motd #
Enter TEXT message.  End with the character '#'.
Unauthorized access is strictly prohibited.#
S2(config)#exit
S2#clock set 20:31:00 22 sep 2026
S2#copy run sta
Destination filename [startup-config]? 
Building configuration...
[OK]
```




