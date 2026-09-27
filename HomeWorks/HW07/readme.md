# Лабораторная работа. Развертывание коммутируемой сети с резервными каналами.

## Топология:
![](Top.jpg)

## Таблица адресации:

|   Устройство   |   Интерфейс   |   IP-адрес        |   Маска подсети  |
|  ------------  |  -----------  |  --------------   | -----------------|
|   S1           |   VLAN 1      |   192.168.1.1     |  255.255.255.0   |
|   S2           |   VLAN 1      |   192.168.1.2     |  255.255.255.0   |
|   S3           |   VLAN 1      |   192.168.1.3     |  255.255.255.0   |

## Цели:

### Часть 1. Создание сети и настройка основных параметров устройства.

### Часть 2. Выбор корневого моста.

### Часть 3. Наблюдение за процессом выбора протоколом STP порта, исходя из стоимости портов.

### Часть 4. Наблюдение за процессом выбора протоколом STP порта, исходя из приоритета портов.

## Решение:

### Часть 1:	Создание сети и настройка основных параметров устройства.

Шаг 1:	Создайте сеть согласно топологии.

Шаг 2:	Выполните инициализацию и перезагрузку коммутаторов.

Шаг 3:	Настройте базовые параметры каждого коммутатора.

* a)	Отключите поиск DNS.

* b)	Присвойте имена устройствам в соответствии с топологией.

* c)	Назначьте class в качестве зашифрованного пароля доступа к привилегированному режиму.

* d)	Назначьте cisco в качестве паролей консоли и VTY и активируйте вход для консоли и VTY каналов.

* e)	Настройте logging synchronous для консольного канала.

* f)	Настройте баннерное сообщение дня (MOTD) для предупреждения пользователей о запрете несанкционированного доступа.

* g)	Задайте IP-адрес, указанный в таблице адресации для VLAN 1 на всех коммутаторах.
  
* h)	Скопируйте текущую конфигурацию в файл загрузочной конфигурации.

```
Switch>en
Switch#conf t
Switch(config)#no ip domain-lookup
Switch(config)#hostname S1
S1(config)#enable secret class
S1(config)#line con 0
S1(config-line)#password cisco
S1(config-line)#login
S1(config-line)#line vty 0 4
S1(config-line)#password cisco
S1(config-line)#login
S1(config-line)#line con 0
S1(config-line)#logging synchronous 
S1(config-line)#exit
S1(config)#service password-encryption 
S1(config)#banner motd #Authorized Access Only!#
S1(config)#int vlan 1
S1(config-if)#ip address 192.168.1.1 255.255.255.0
S1(config-if)#no sh
S1(config)#end
S1#copy run sta
Destination filename [startup-config]? 
Building configuration...
[OK]
```

```
Switch>en
Switch#conf t
Switch(config)#no ip domain-lookup
Switch(config)#hostname S2
S2(config)#enable secret class
S2(config-line)#line con 0
S2(config-line)#password cisco
S2(config-line)#login
S2(config-line)#logging synchronous 
S2(config-line)#line vty 0 4
S2(config-line)#passwor cisco
S2(config-line)#login
S2(config-line)#exit
S2(config)#service password-encryption 
S2(config)#banner motd #Authorized Access Only!#
S2(config)#int vlan 1
S2(config-if)#ip address 192.168.1.2 255.255.255.0
S2(config-if)#no sh
S2(config-if)#end
S2#copy run sta
Destination filename [startup-config]? 
Building configuration...
[OK]
```

```
Switch>en
Switch#conf t
Switch(config)#no ip domain-lookup
Switch(config)#hostname S3
S3(config)#enable secret class
S3(config)#line con 0
S3(config-line)#password cisco
S3(config-line)#login
S3(config-line)#logging synchronous
S3(config-line)#line vty 0 4
S3(config-line)#passwor cisco
S3(config-line)#login
S3(config-line)#exit
S3(config)#service password-encryption 
S3(config)#banner motd #Authorized Access Only!#
S3(config)#int vlan 1
S3(config-if)#ip address 192.168.1.3 255.255.255.0
S3(config-if)#no sh
S3(config-if)#end
S3#copy run sta
Destination filename [startup-config]? 
Building configuration...
[OK]
```

Шаг 4:	Проверьте связь.

```
S1#ping 192.168.1.2

Type escape sequence to abort.
Sending 5, 100-byte ICMP Echos to 192.168.1.2, timeout is 2 seconds:
!!!!!
Success rate is 100 percent (5/5), round-trip min/avg/max = 0/0/0 ms

S1#ping 192.168.1.3

Type escape sequence to abort.
Sending 5, 100-byte ICMP Echos to 192.168.1.3, timeout is 2 seconds:
!!!!!
Success rate is 100 percent (5/5), round-trip min/avg/max = 0/0/0 ms
```

```
S2#ping 192.168.1.3

Type escape sequence to abort.
Sending 5, 100-byte ICMP Echos to 192.168.1.3, timeout is 2 seconds:
!!!!!
Success rate is 100 percent (5/5), round-trip min/avg/max = 0/0/0 ms
```

### Часть 2. Выбор корневого моста.

Шаг 1:	Отключите все порты на коммутаторах.

```
S1(config)#int range f0/1 - 24, g0/1 - 2
S1(config-if-range)#shutdown 
```

```
S2(config)#int range f0/1 - 24, g0/1 - 2
S2(config-if-range)#shutdown
```

```
S3(config)#int range f0/1 - 24, g0/1 - 2
S3(config-if-range)#shutdown 
```

Шаг 2:	Настройте подключенные порты в качестве транковых.

```
S1(config)#int f0/1
S1(config-if)#switchport mode trunk
S1(config-if)#switchport trunk native vlan 1
S1(config)#int f0/2
S1(config-if)#switchport mode trunk
S1(config-if)#switchport trunk native vlan 1
S1(config)#int f0/3
S1(config-if)#switchport mode trunk
S1(config-if)#switchport trunk native vlan 1
S1(config-if)#int f0/4
S1(config-if)#switchport mode trunk
S1(config-if)#switchport trunk native vlan 1
```

```
S2(config)#int f0/1
S2(config-if)#switchport mode trunk 
S2(config-if)#switchport trunk native vlan 1
S2(config-if)#int f0/2
S2(config-if)#switchport mode trunk 
S2(config-if)#switchport trunk native vlan 1
S2(config-if)#int f0/3
S2(config-if)#switchport mode trunk 
S2(config-if)#switchport trunk native vlan 1
S2(config-if)#int f0/4
S2(config-if)#switchport mode trunk 
S2(config-if)#switchport trunk native vlan 1
```

```
S3(config)#int f0/1
S3(config-if)#switchport mode trunk 
S3(config-if)#switchport trunk native vlan 1
S3(config-if)#int f0/2
S3(config-if)#switchport mode trunk 
S3(config-if)#switchport trunk native vlan 1
S3(config-if)#int f0/3
S3(config-if)#switchport mode trunk 
S3(config-if)#switchport trunk native vlan 1
S3(config-if)#int f0/4
S3(config-if)#switchport mode trunk 
S3(config-if)#switchport trunk native vlan 1
```

Шаг 3:	Включите порты F0/2 и F0/4 на всех коммутаторах.

```
S1(config)#int f0/2
S1(config-if)#no sh
S1(config-if)#int f0/4
S1(config-if)#no sh
```

```
S2(config-if)#int f0/2
S2(config-if)#no sh
S2(config-if)#int f0/4
S2(config-if)#no sh
```

```
S3(config-if)#int f0/2
S3(config-if)#no sh
S3(config-if)#int f0/4
S3(config-if)#no sh
```

Шаг 4:	Отобразите данные протокола spanning-tree.

```
S1#sh spanning-tree 
VLAN0001
  Spanning tree enabled protocol ieee
  Root ID    Priority    32769
             Address     0001.9729.0709
             Cost        19
             Port        4(FastEthernet0/4)
             Hello Time  2 sec  Max Age 20 sec  Forward Delay 15 sec

  Bridge ID  Priority    32769  (priority 32768 sys-id-ext 1)
             Address     0090.0C3E.82BB
             Hello Time  2 sec  Max Age 20 sec  Forward Delay 15 sec
             Aging Time  20

Interface        Role Sts Cost      Prio.Nbr Type
---------------- ---- --- --------- -------- --------------------------------
Fa0/2            Altn BLK 19        128.2    P2p
Fa0/4            Root FWD 19        128.4    P2p
```

```
S2#sh spanning-tree 
VLAN0001
  Spanning tree enabled protocol ieee
  Root ID    Priority    32769
             Address     0001.9729.0709
             Cost        19
             Port        4(FastEthernet0/4)
             Hello Time  2 sec  Max Age 20 sec  Forward Delay 15 sec

  Bridge ID  Priority    32769  (priority 32768 sys-id-ext 1)
             Address     0001.9765.60E6
             Hello Time  2 sec  Max Age 20 sec  Forward Delay 15 sec
             Aging Time  20

Interface        Role Sts Cost      Prio.Nbr Type
---------------- ---- --- --------- -------- --------------------------------
Fa0/4            Root FWD 19        128.4    P2p
Fa0/2            Desg FWD 19        128.2    P2p
```

```
S3#sh spanning-tree 
VLAN0001
  Spanning tree enabled protocol ieee
  Root ID    Priority    32769
             Address     0001.9729.0709
             This bridge is the root
             Hello Time  2 sec  Max Age 20 sec  Forward Delay 15 sec

  Bridge ID  Priority    32769  (priority 32768 sys-id-ext 1)
             Address     0001.9729.0709
             Hello Time  2 sec  Max Age 20 sec  Forward Delay 15 sec
             Aging Time  20

Interface        Role Sts Cost      Prio.Nbr Type
---------------- ---- --- --------- -------- --------------------------------
Fa0/2            Desg FWD 19        128.2    P2p
Fa0/4            Desg FWD 19        128.4    P2p
```

С учетом выходных данных, поступающих с коммутаторов, ответьте на следующие вопросы.

Какой коммутатор является корневым мостом? - *Корневым мостом выбран коммутатор S3.*

Почему этот коммутатор был выбран протоколом spanning-tree в качестве корневого моста? - *Потому что у коммутатора S3 наименьший BID.*
*Приоритет мостов и расширенный идентификатор у всех коммутатров одинаковый, значит выбор будет происходит по наименьшему MAC-адресу.*

Какие порты на коммутаторе являются корневыми портами? - *На S1 - Fa0/4, на S2 - Fa0/4* 

Какие порты на коммутаторе являются назначенными портами? - *На S2 - Fa0/2, на S3 - Fa0/2, Fa0/4*

Какой порт отображается в качестве альтернативного и в настоящее время заблокирован? - *S1 - Fa0/2*

Почему протокол spanning-tree выбрал этот порт в качестве невыделенного (заблокированного) порта? - *Потому что стоимость пути через fa0/2 выше.*

### Часть 3:	Наблюдение за процессом выбора протоколом STP порта, исходя из стоимости портов.

Шаг 1:	Определите коммутатор с заблокированным портом.

```
S1#sh spanning-tree 
VLAN0001
  Spanning tree enabled protocol ieee
  Root ID    Priority    32769
             Address     0001.9729.0709
             Cost        19
             Port        4(FastEthernet0/4)
             Hello Time  2 sec  Max Age 20 sec  Forward Delay 15 sec

  Bridge ID  Priority    32769  (priority 32768 sys-id-ext 1)
             Address     0090.0C3E.82BB
             Hello Time  2 sec  Max Age 20 sec  Forward Delay 15 sec
             Aging Time  20

Interface        Role Sts Cost      Prio.Nbr Type
---------------- ---- --- --------- -------- --------------------------------
Fa0/2            Altn BLK 19        128.2    P2p
Fa0/4            Root FWD 19        128.4    P2p
```

```
S2#sh spanning-tree 
VLAN0001
  Spanning tree enabled protocol ieee
  Root ID    Priority    32769
             Address     0001.9729.0709
             Cost        19
             Port        4(FastEthernet0/4)
             Hello Time  2 sec  Max Age 20 sec  Forward Delay 15 sec

  Bridge ID  Priority    32769  (priority 32768 sys-id-ext 1)
             Address     0001.9765.60E6
             Hello Time  2 sec  Max Age 20 sec  Forward Delay 15 sec
             Aging Time  20

Interface        Role Sts Cost      Prio.Nbr Type
---------------- ---- --- --------- -------- --------------------------------
Fa0/4            Root FWD 19        128.4    P2p
Fa0/2            Desg FWD 19        128.2    P2p
```

Шаг 2:	Измените стоимость порта.

```
S1(config)#int f0/4
S1(config-if)#spanning-tree vlan 1 cost 18
```

Шаг 3:	Просмотрите изменения протокола spanning-tree.

```
S1#sh spanning-tree 
VLAN0001
  Spanning tree enabled protocol ieee
  Root ID    Priority    32769
             Address     0001.9729.0709
             Cost        18
             Port        4(FastEthernet0/4)
             Hello Time  2 sec  Max Age 20 sec  Forward Delay 15 sec

  Bridge ID  Priority    32769  (priority 32768 sys-id-ext 1)
             Address     0090.0C3E.82BB
             Hello Time  2 sec  Max Age 20 sec  Forward Delay 15 sec
             Aging Time  20

Interface        Role Sts Cost      Prio.Nbr Type
---------------- ---- --- --------- -------- --------------------------------
Fa0/2            Desg FWD 19        128.2    P2p
Fa0/4            Root FWD 18        128.4    P2p
```

```
S2#sh spanning-tree 
VLAN0001
  Spanning tree enabled protocol ieee
  Root ID    Priority    32769
             Address     0001.9729.0709
             Cost        19
             Port        4(FastEthernet0/4)
             Hello Time  2 sec  Max Age 20 sec  Forward Delay 15 sec

  Bridge ID  Priority    32769  (priority 32768 sys-id-ext 1)
             Address     0001.9765.60E6
             Hello Time  2 sec  Max Age 20 sec  Forward Delay 15 sec
             Aging Time  20

Interface        Role Sts Cost      Prio.Nbr Type
---------------- ---- --- --------- -------- --------------------------------
Fa0/4            Root FWD 19        128.4    P2p
Fa0/2            Altn BLK 19        128.2    P2p
```

Почему протокол spanning-tree заменяет ранее заблокированный порт на назначенный порт и блокирует порт, который был назначенным портом на другом коммутаторе? - 
*Изменение происходит из-за снижения стоимости порта*

Шаг 4:	Удалите изменения стоимости порта.

```
S1(config)#int f0/4
S1(config-if)#no spanning-tree vlan 1 cost 18
```

```
S1#sh spanning-tree 
VLAN0001
  Spanning tree enabled protocol ieee
  Root ID    Priority    32769
             Address     0001.9729.0709
             Cost        19
             Port        4(FastEthernet0/4)
             Hello Time  2 sec  Max Age 20 sec  Forward Delay 15 sec

  Bridge ID  Priority    32769  (priority 32768 sys-id-ext 1)
             Address     0090.0C3E.82BB
             Hello Time  2 sec  Max Age 20 sec  Forward Delay 15 sec
             Aging Time  20

Interface        Role Sts Cost      Prio.Nbr Type
---------------- ---- --- --------- -------- --------------------------------
Fa0/2            Altn BLK 19        128.2    P2p
Fa0/4            Root FWD 19        128.4    P2p
```

```
S2#sh spanning-tree 
VLAN0001
  Spanning tree enabled protocol ieee
  Root ID    Priority    32769
             Address     0001.9729.0709
             Cost        19
             Port        4(FastEthernet0/4)
             Hello Time  2 sec  Max Age 20 sec  Forward Delay 15 sec

  Bridge ID  Priority    32769  (priority 32768 sys-id-ext 1)
             Address     0001.9765.60E6
             Hello Time  2 sec  Max Age 20 sec  Forward Delay 15 sec
             Aging Time  20

Interface        Role Sts Cost      Prio.Nbr Type
---------------- ---- --- --------- -------- --------------------------------
Fa0/4            Root FWD 19        128.4    P2p
Fa0/2            Desg FWD 19        128.2    P2p
```

### Часть 4:	Наблюдение за процессом выбора протоколом STP порта, исходя из приоритета портов.

```
S1#sh spanning-tree 
VLAN0001
  Spanning tree enabled protocol ieee
  Root ID    Priority    32769
             Address     0001.9729.0709
             Cost        19
             Port        3(FastEthernet0/3)
             Hello Time  2 sec  Max Age 20 sec  Forward Delay 15 sec

  Bridge ID  Priority    32769  (priority 32768 sys-id-ext 1)
             Address     0090.0C3E.82BB
             Hello Time  2 sec  Max Age 20 sec  Forward Delay 15 sec
             Aging Time  20

Interface        Role Sts Cost      Prio.Nbr Type
---------------- ---- --- --------- -------- --------------------------------
Fa0/3            Root FWD 19        128.3    P2p
Fa0/1            Altn BLK 19        128.1    P2p
Fa0/2            Altn BLK 19        128.2    P2p
Fa0/4            Altn BLK 19        128.4    P2p
```

```
S2#sh spanning-tree 
VLAN0001
  Spanning tree enabled protocol ieee
  Root ID    Priority    32769
             Address     0001.9729.0709
             Cost        19
             Port        3(FastEthernet0/3)
             Hello Time  2 sec  Max Age 20 sec  Forward Delay 15 sec

  Bridge ID  Priority    32769  (priority 32768 sys-id-ext 1)
             Address     0001.9765.60E6
             Hello Time  2 sec  Max Age 20 sec  Forward Delay 15 sec
             Aging Time  20

Interface        Role Sts Cost      Prio.Nbr Type
---------------- ---- --- --------- -------- --------------------------------
Fa0/4            Altn BLK 19        128.4    P2p
Fa0/2            Desg FWD 19        128.2    P2p
Fa0/3            Root FWD 19        128.3    P2p
Fa0/1            Desg FWD 19        128.1    P2p
```

```
S3#sh spanning-tree 
VLAN0001
  Spanning tree enabled protocol ieee
  Root ID    Priority    32769
             Address     0001.9729.0709
             This bridge is the root
             Hello Time  2 sec  Max Age 20 sec  Forward Delay 15 sec

  Bridge ID  Priority    32769  (priority 32768 sys-id-ext 1)
             Address     0001.9729.0709
             Hello Time  2 sec  Max Age 20 sec  Forward Delay 15 sec
             Aging Time  20

Interface        Role Sts Cost      Prio.Nbr Type
---------------- ---- --- --------- -------- --------------------------------
Fa0/1            Desg FWD 19        128.1    P2p
Fa0/3            Desg FWD 19        128.3    P2p
Fa0/2            Desg FWD 19        128.2    P2p
Fa0/4            Desg FWD 19        128.4    P2p
```

Какой порт выбран протоколом STP в качестве порта корневого моста на каждом коммутаторе некорневого моста? - *На S1 - Fa0/3, на S2 - Fa0/3*

Почему протокол STP выбрал эти порты в качестве портов корневого моста на этих коммутаторах? - *Порты выбраны исходя из Port ID. Т.к. стоимость пути и BID совпадают.*


### Вопросы для повторения

1.	Какое значение протокол STP использует первым после выбора корневого моста, чтобы определить выбор порта? - *После выбора корневого моста протокол STP определяет стоимость пути до корневого моста, чтобы выбрать корневой порт на каждом некорневом коммутаторе.*

2.	Если первое значение на двух портах одинаково, какое следующее значение будет использовать протокол STP при выборе порта? - *Следующим значением будет Bridge ID.*

3.	Если оба значения на двух портах равны, каким будет следующее значение, которое использует протокол STP при выборе порта? - *Следующим значением будет Port ID.*
