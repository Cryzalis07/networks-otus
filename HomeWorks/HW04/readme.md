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

### Часть 2. Ручная настройка IPv6-адресов.

Шаг 1. Назначим IPv6-адреса интерфейсам Ethernet на R1.

*  a)	Назначим глобальные индивидуальные IPv6-адреса, указанные в таблице адресации обоим интерфейсам Ethernet на R1.

```
Router>en
Router#conf t
R1(config)#int g0/0/0
R1(config-if)#ipv6 address 2001:db8:acad:a::1/64
R1(config-if)#no sh

R1(config-if)#
%LINK-5-CHANGED: Interface GigabitEthernet0/0/0, changed state to up

%LINEPROTO-5-UPDOWN: Line protocol on Interface GigabitEthernet0/0/0, changed state to up

R1(config-if)#int g0/0/1
R1(config-if)#ipv6 address 2001:db8:acad:1::1/64
R1(config-if)#no sh

R1(config-if)#
%LINK-5-CHANGED: Interface GigabitEthernet0/0/1, changed state to up

%LINEPROTO-5-UPDOWN: Line protocol on Interface GigabitEthernet0/0/1, changed state to up

```

* b)	Введем команду show ipv6 interface brief, чтобы проверить, назначен ли каждому интерфейсу корректный индивидуальный IPv6-адрес.

```
R1#sh ipv6 int br
GigabitEthernet0/0/0       [up/up]
    FE80::202:16FF:FEC6:7B01
    2001:DB8:ACAD:A::1
GigabitEthernet0/0/1       [up/up]
    FE80::202:16FF:FEC6:7B02
    2001:DB8:ACAD:1::1
Vlan1                      [administratively down/down]
    unassigned
```

* c)	Чтобы обеспечить соответствие локальных адресов канала индивидуальному адресу, вручную введем локальные адреса канала на каждом интерфейсе Ethernet на R1.

```
R1(config)#int g0/0/0
R1(config-if)#ipv6 address fe80::1 link-local
R1(config-if)#int g0/0/1
R1(config-if)#ipv6 address fe80::1 link-local
```

* d)	Убедимся, что локальный адрес связи изменен на fe80::1.

```
R1#sh ipv6 int br
GigabitEthernet0/0/0       [up/up]
    FE80::1
    2001:DB8:ACAD:A::1
GigabitEthernet0/0/1       [up/up]
    FE80::1
    2001:DB8:ACAD:1::1
Vlan1                      [administratively down/down]
    unassigned
```

Шаг 2. Активируем IPv6-маршрутизацию на R1.

* a)	В командной строке на PC-B введем команду ipconfig, чтобы получить данные IPv6-адреса, назначенного интерфейсу ПК.

```
C:\>ipconfig

FastEthernet0 Connection:(default port)

   Connection-specific DNS Suffix..: 
   Link-local IPv6 Address.........: FE80::20A:F3FF:FE3E:D630
   IPv6 Address....................: ::
   IPv4 Address....................: 0.0.0.0
   Subnet Mask.....................: 0.0.0.0
   Default Gateway.................: ::
                                     0.0.0.0
```

* b)	Активируем IPv6-маршрутизацию на R1 с помощью команды IPv6 unicast-routing.

```
R1(config)#ipv6 unicast-routing

```

* c)	Теперь, когда R1 входит в группу многоадресной рассылки всех маршрутизаторов, еще раз введите команду ipconfig на PC-B. Проверьте данные IPv6-адреса.

```
C:\>ipconfig

FastEthernet0 Connection:(default port)

   Connection-specific DNS Suffix..: 
   Link-local IPv6 Address.........: FE80::20A:F3FF:FE3E:D630
   IPv6 Address....................: 2001:DB8:ACAD:A:20A:F3FF:FE3E:D630
   IPv4 Address....................: 0.0.0.0
   Subnet Mask.....................: 0.0.0.0
   Default Gateway.................: FE80::1
                                     0.0.0.0
```

Почему PC-B получил глобальный префикс маршрутизации и идентификатор подсети, которые вы настроили на R1?

Потому что эту информацию PC-B получил в RA от R1.

Шаг 3. Назначим IPv6-адреса интерфейсу управления (SVI) на S1.

* a)	Назначим адрес IPv6 для S1. Также назначим этому интерфейсу локальный адрес канала fe80::b.

```
S1(config)#int vlan 1
S1(config-if)#ipv6 address 2001:db8:acad:1::b/64
S1(config-if)#ipv6 address fe80::b link-local
S1(config-if)#no sh
```

* b)	Проверим правильность назначения IPv6-адресов интерфейсу управления с помощью команды show ipv6 interface vlan1.

```
S1#sh ipv6 int vlan 1
Vlan1 is up, line protocol is up
  IPv6 is enabled, link-local address is FE80::B
  No Virtual link-local address(es):
  Global unicast address(es):
    2001:DB8:ACAD:1::B, subnet is 2001:DB8:ACAD:1::/64
  Joined group address(es):
    FF02::1
    FF02::1:FF00:B
  MTU is 1500 bytes
  ICMP error messages limited to one every 100 milliseconds
  ICMP redirects are enabled
  ICMP unreachables are sent
  Output features: Check hwidb
  ND DAD is enabled, number of DAD attempts: 1
  ND reachable time is 30000 milliseconds
```

Шаг 4. Назначим компьютерам статические IPv6-адреса.

### Часть 3. Проверка сквозного подключения

