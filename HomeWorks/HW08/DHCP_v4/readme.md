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
|  S2            |  VLAN1        |  192.168.1.98     | 255.255.255.240  | 192.168.1.97         |
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

### Часть 2. Настройка и проверка двух серверов DHCPv4 на R1.

### Часть 3. Настройка и проверка DHCP-ретрансляции на R2.

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
R1(config-if)#int g0/0/1.100
R1(config-subif)#encapsulation dot1Q 100
R1(config-subif)#ip address 192.168.1.1 255.255.255.192
R1(config-subif)#int g0/0/1.200
R1(config-subif)#encapsulation dot1Q 200
R1(config-subif)#ip address 192.168.1.65 255.255.255.224
R1(config-subif)#int g0/0/1.1000
R1(config-subif)#encapsulation dot1Q 1000 native 
R1#sh ip int br
Interface              IP-Address      OK? Method Status                Protocol 
GigabitEthernet0/0/0   unassigned      YES unset  administratively down down 
GigabitEthernet0/0/1   unassigned      YES unset  up                    up 
GigabitEthernet0/0/1.100192.168.1.1     YES manual up                    up 
GigabitEthernet0/0/1.200192.168.1.65    YES manual up                    up 
GigabitEthernet0/0/1.1000unassigned      YES unset  up                    up 
Vlan1                  unassigned      YES unset  administratively down down
```

Шаг 5.	Настроим G0/0/1 на R2, затем G0/0/0 и статическую маршрутизацию для обоих маршрутизаторов.

```
R2>en
R2#conf t
R2(config)#int g0/0/1
R2(config-if)#ip address 192.168.1.97 255.255.255.240
R2(config-if)#no sh
R2(config)#int g0/0/0
R2(config-if)#ip ad
R2(config-if)#ip address 10.0.0.2 255.255.255.252
R2(config-if)#no sh
R2(config)#ip route 0.0.0.0 0.0.0.0 10.0.0.1
R2#copy running-config startup-config 
Destination filename [startup-config]? 
Building configuration...
[OK]
```

```
R1(config)#int g0/0/0
R1(config-if)#ip ad
R1(config-if)#ip address 10.0.0.1 255.255.255.252
R1(config-if)#no sh
R1(config)#ip route 0.0.0.0 0.0.0.0 10.0.0.2
R1#ping 192.168.1.97

Type escape sequence to abort.
Sending 5, 100-byte ICMP Echos to 192.168.1.97, timeout is 2 seconds:
!!!!!
Success rate is 100 percent (5/5), round-trip min/avg/max = 0/0/0 ms
R1#copy running-config startup-config 
Destination filename [startup-config]? 
Building configuration...
```

Шаг 6.	Настроим базовые параметры каждого коммутатора.

```
Switch>en
Switch#conf t
Switch(config)#no ip domain-lookup
Switch(config)#hostname S1
S1(config)#enable secret class
S1(config)#line con 0
S1(config-line)#password cisco
S1(config-line)#login
S1(config)#line vty 0 4
S1(config-line)#password cisco
S1(config-line)#login
S1(config-line)#exit
S1(config)#service password-encryption 
S1(config)#banner motd #Authorized Access Only!#
S1#copy running-config startup-config 
Destination filename [startup-config]? 
Building configuration...
[OK]
S1#clock set 22:13:00 22 oct 2026
S1#copy running-config startup-config 
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
S2#copy running-config startup-config 
Destination filename [startup-config]? 
Building configuration...
[OK]
S2#clock set 22:22:00 22 oct 2026
S2#copy running-config startup-config 
Destination filename [startup-config]? 
Building configuration...
[OK]
```

Шаг 7.	Создадим сети VLAN на коммутаторе S1.

```
S1(config)#vlan 100
S1(config-vlan)#name Clients
S1(config-vlan)#vlan 200
S1(config-vlan)#name Management
S1(config-vlan)#vlan 999
S1(config-vlan)#name Perking_lot
S1(config-vlan)#vlan 1000
S1(config-vlan)#name Native
S1(config-vlan)#int vlan 200
S1(config-if)#ip address 192.168.1.66 255.255.255.224
S1(config-if)#exit
S1(config)#ip default-gateway 192.168.1.65
S1(config)#int range f0/1-4, f0/7-24, g0/1-2
S1(config-if-range)#sw mo ac
S1(config-if-range)#sw ac vlan 999
S1(config-if-range)#sh
```

```
S2>en
S2#conf t
S2(config)#int vlan 1
S2(config-if)#ip address 192.168.1.98 255.255.255.240
S2(config-if)#exit
S2(config)#ip default-gateway 192.168.1.97
S2(config)#int range f0/1-4, f0/6-17, f0/19-24, g0/1-2
S2(config-if-range)#sh
```

Шаг 8.	Назначим сети VLAN соответствующим интерфейсам коммутатора.

```
S1(config-if)#int f0/6
S1(config-if)#sw mo ac
S1(config-if)#sw ac vlan 100
```

Шаг 9.	Вручную настроим интерфейс S1 F0/5 в качестве транка 802.1Q.

```
S1(config-if)#int f0/5
S1(config-if)#sw mo tr
S1(config-if)#sw trunk allowed vlan 100,200,1000
S1#copy run sta
Destination filename [startup-config]? 
Building configuration...
[OK]
S1#sh int trunk 
Port        Mode         Encapsulation  Status        Native vlan
Fa0/5       on           802.1q         trunking      1000

Port        Vlans allowed on trunk
Fa0/5       100,200,1000

Port        Vlans allowed and active in management domain
Fa0/5       100,200,1000

Port        Vlans in spanning tree forwarding state and not pruned
Fa0/5       100,200,1000
```

### Часть 2. Настройка и проверка двух серверов DHCPv4 на R1.

Шаг 1.	Настроим R1 с пулами DHCPv4 для двух поддерживаемых подсетей. Ниже приведен только пул DHCP для подсети A.

```
R1(config)#ip dhcp excluded-address 192.168.1.1 192.168.1.5
R1(config)#ip dhcp pool R1_Client_LAN
R1(dhcp-config)#network 192.168.1.0 255.255.255.192
R1(dhcp-config)#default-router 192.168.1.1
R1(dhcp-config)#domain-name CCNA-lab.com
R1(config)#ip dhcp excluded-address 192.168.1.97 192.168.1.101
R1(config)#ip dhcp pool R2_Client_LAN
R1(dhcp-config)#network 192.168.1.96 255.255.255.240
R1(dhcp-config)#default-router 192.168.1.97
R1(dhcp-config)#domain-name CCNA-lab.com
```

Шаг 2.	Сохраним конфигурацию.

```
R1#copy run sta
Destination filename [startup-config]? 
Building configuration...
[OK]
```

Шаг 3.	Проверка конфигурации сервера DHCPv4.

```
R1#show ip dhcp pool

Pool R1_Client_LAN :
 Utilization mark (high/low)    : 100 / 0
 Subnet size (first/next)       : 0 / 0 
 Total addresses                : 62
 Leased addresses               : 1
 Excluded addresses             : 2
 Pending event                  : none

 1 subnet is currently in the pool
 Current index        IP address range                    Leased/Excluded/Total
 192.168.1.1          192.168.1.1      - 192.168.1.62      1    / 2     / 62

Pool R2_Client_LAN :
 Utilization mark (high/low)    : 100 / 0
 Subnet size (first/next)       : 0 / 0 
 Total addresses                : 14
 Leased addresses               : 0
 Excluded addresses             : 2
 Pending event                  : none

 1 subnet is currently in the pool
 Current index        IP address range                    Leased/Excluded/Total
 192.168.1.97         192.168.1.97     - 192.168.1.110     0    / 2     / 14
R1#show ip dhcp binding
IP address       Client-ID/              Lease expiration        Type
                 Hardware address
192.168.1.6      0005.5E51.C996           --                     Automatic
```

Шаг 4.	Попытка получить IP-адрес от DHCP на PC-A.

```
C:\>ipconfig /all

FastEthernet0 Connection:(default port)

   Connection-specific DNS Suffix..: CCNA-lab.com
   Physical Address................: 0005.5E51.C996
   Link-local IPv6 Address.........: FE80::205:5EFF:FE51:C996
   IPv6 Address....................: ::
   IPv4 Address....................: 192.168.1.6
   Subnet Mask.....................: 255.255.255.192
   Default Gateway.................: ::
                                     192.168.1.1
   DHCP Servers....................: 192.168.1.1
   DHCPv6 IAID.....................: 
   DHCPv6 Client DUID..............: 00-01-00-01-30-06-5C-AB-00-05-5E-51-C9-96
   DNS Servers.....................: ::
                                     0.0.0.0

C:\>ping 192.168.1.97

Pinging 192.168.1.97 with 32 bytes of data:

Request timed out.
Reply from 192.168.1.97: bytes=32 time<1ms TTL=254
Reply from 192.168.1.97: bytes=32 time<1ms TTL=254
Reply from 192.168.1.97: bytes=32 time<1ms TTL=254

Ping statistics for 192.168.1.97:
    Packets: Sent = 4, Received = 3, Lost = 1 (25% loss),
Approximate round trip times in milli-seconds:
    Minimum = 0ms, Maximum = 0ms, Average = 0ms
```

### Часть 3. Настройка и проверка DHCP-ретрансляции на R2.

Шаг 1.	Настройка R2 в качестве агента DHCP-ретрансляции для локальной сети на G0/0/1

```
R2(config)#int g0/0/1
R2(config-if)#ip helper-address 10.0.0.1
R2#copy run sta
Destination filename [startup-config]? 
Building configuration...
[OK]
```

Шаг 2.	Попытка получить IP-адрес от DHCP на PC-B.

```
C:\>ipconfig /renew

   IP Address......................: 192.168.1.102
   Subnet Mask.....................: 255.255.255.240
   Default Gateway.................: 192.168.1.97
   DNS Server......................: 0.0.0.0

C:\>ipconfig /all

FastEthernet0 Connection:(default port)

   Connection-specific DNS Suffix..: CCNA-lab.com
   Physical Address................: 000A.F372.5D64
   Link-local IPv6 Address.........: FE80::20A:F3FF:FE72:5D64
   IPv6 Address....................: ::
   IPv4 Address....................: 192.168.1.102
   Subnet Mask.....................: 255.255.255.240
   Default Gateway.................: ::
                                     192.168.1.97
   DHCP Servers....................: 10.0.0.1
   DHCPv6 IAID.....................: 
   DHCPv6 Client DUID..............: 00-01-00-01-17-ED-30-A9-00-0A-F3-72-5D-64
   DNS Servers.....................: ::
                                     0.0.0.0

C:\>ping 192.168.1.1

Pinging 192.168.1.1 with 32 bytes of data:

Reply from 192.168.1.1: bytes=32 time<1ms TTL=254
Reply from 192.168.1.1: bytes=32 time<1ms TTL=254
Reply from 192.168.1.1: bytes=32 time<1ms TTL=254
Reply from 192.168.1.1: bytes=32 time<1ms TTL=254

Ping statistics for 192.168.1.1:
    Packets: Sent = 4, Received = 4, Lost = 0 (0% loss),
Approximate round trip times in milli-seconds:
    Minimum = 0ms, Maximum = 0ms, Average = 0ms
```

```
R1#show ip dhcp binding
IP address       Client-ID/              Lease expiration        Type
                 Hardware address
192.168.1.6      0005.5E51.C996           --                     Automatic
192.168.1.102    000A.F372.5D64           --                     Automatic


```

