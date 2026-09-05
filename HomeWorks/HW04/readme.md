# Лабораторная работа. Настройка IPv6-адресов на сетевых устройствах

## Топология:
![](Top.jpg)

## Таблица адресации:

|   Устройство   |   Интерфейс   |   IPv6-адрес        |   Link local IPv6-адрес  |  Длина префикса   |  Шлюз по умолчанию   |
|  ------------  |  -----------  |  ----------------   | -------------------------|  ---------------- |  -----------------   |
|  R1            |  G0/0/0       |  2001:db8:acad:a::1 |   fe80::1                |  64               |    -                 |
|  R1            |  G0/0/1       |  2001:db8:acad:1::1 |   fe80::1                |  64               |    -                 |
|  S1            |  VLAN 1       |  2001:db8:acad:1::b |   fe80::b                |  64               |    -                 |
|  PC-A          |  NIC          |  2001:db8:acad:1::3 |   SLACC                  |  64               |    fe80::1           |
|  PC-B          |  NIC          |  2001:db8:acad:a::3 |   SLACC                  |  64               |    fe80::1           |

## Задачи:

### Часть 1. Настройка топологии и конфигурация основных параметров маршрутизатора и коммутатора.

### Часть 2. Ручная настройка IPv6-адресов.

### Часть 3. Проверка сквозного соединения.

## Решение:

### Часть 1. Настройка топологии и конфигурация основных параметров маршрутизатора и коммутатора.

Построим сеть согласно топологии в Cisco Packet Tracer.

Шаг 1. Настроим маршрутизатор.

Назначим имя хоста и настроим основные параметры устройства.

```
Router>en
Router#conf t
Router(config)#hostname R1
R1(config)#enable secret class
R1(config)#service password-encryption
R1(config)#line con 0
R1(config-line)#password cisco
R1(config-line)#login
R1(config-line)#line vty 0 4
R1(config-line)#password cisco
R1(config-line)#login
R1(config-line)#end
R1#copy run sta
Destination filename [startup-config]? 
Building configuration...
[OK]
```

Шаг 2. Настроим коммутатор.

Перед настройкой проверим режим поддержки IPv6 коммутатором. 
Переключим коммутатор на шаблон работы IPv4 и IPv6.
Назначем имя хоста и настроим основные параметры устройства.

```
Switch>en
Switch#show sdm prefer
 The current template is "default" template.
 The selected template optimizes the resources in
 the switch to support this level of features for
 0 routed interfaces and 1024 VLANs.

  number of unicast mac addresses:                  8K
  number of IPv4 IGMP groups + multicast routes:    0.25K
  number of IPv4 unicast routes:                    0
  number of IPv6 multicast groups:                  0
  number of directly-connected IPv6 addresses:      0
  number of indirect IPv6 unicast routes:           0
  number of IPv4 policy based routing aces:         0
  number of IPv4/MAC qos aces:                      0.125k
  number of IPv4/MAC security aces:                 0.375k
  number of IPv6 policy based routing aces:         0
  number of IPv6 qos aces:                          20
  number of IPv6 security aces:                     25

Switch#conf t
Switch(config)#sdm prefer dual-ipv4-and-ipv6 default
Changes to the running SDM preferences have been stored, but cannot take effect until the next reload.
Use 'show sdm prefer' to see what SDM preference is currently active.
Switch(config)#end
Switch#reload
System configuration has been modified. Save? [yes/no]:y
Building configuration...
[OK]
Switch>en
Switch#conf t
Switch(config)#hostname S1
S1(config)#enable secret class
S1(config)#service password-encryption
S1(config)#line con 0
S1(config-line)#password cisco
S1(config-line)#login
S1(config-line)#line vty 0 4
S1(config-line)#password cisco
S1(config-line)#login
S1(config-line)#end
S1#copy run sta
Destination filename [startup-config]? 
Building configuration...
[OK]
```

