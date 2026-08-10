# Лабораторная работа. Базовая настройка коммутатора

### Топология
![](Top.png)

## Таблица адресации

|   Устройство   |   Интерфейс   |   IP-адрес / префикс   |
|  ------------  |  -----------  |  --------------------  |
|   S1           |   VLAN 1      |   192.168.1.2 / 24     |
|   PC-A         |   NIC         |   192.168.1.10 / 24    |

## Задачи
### Часть 1. Проверка конфигурации коммутатора по умолчанию

### Часть 2. Создание сети и настройка основных параметров устройства
-	Настроить базовые параметры коммутатора.
- Настроить IP-адрес для ПК.

### Часть 3. Проверка сетевых подключений
-	Отобразите конфигурацию устройства.
-	Протестируйте сквозное соединение, отправив эхо-запрос.
-	Протестируйте возможности удаленного управления с помощью Telnet.

## Решение

### Часть 1. Создание сети и проверка настроек коммутатора по умолчанию.
               
  Шаг 1. Создаем сеть согласно топологии.

  *    a) Подсоединяем консольный кабель, как показано в топологии. На данном этапе не подключаем кабель Ethernet к компьютеру PC-A.

  *    b) Устанавливаем консольное подключение к коммутатору с компьютера PC-A с помощью Terminal или CLI
      
**Почему нужно использовать консольное подключение для первоначальной настройки коммутатора? Почему нельзя подключиться к коммутатору через Telnet или SSH?**

*Первоначальная настройка возможна только через консольное соединение. Другие способы просто недоступны.*

Шаг 2. Проверка настройки коммутатора по умолчанию.

* a) Переходим в привилегированный режим EXEC командой:
```
Switch>enable
```
* b) Просматриваем текущую конфигурацию командой:
```
Switch#show running-config
```
**Сколько интерфейсов FastEthernet имеется на коммутаторе 2960?** -- *24*

**Сколько интерфейсов Gigabit Ethernet имеется на коммутаторе 2960?** -- *2*

**Каков диапазон значений, отображаемых в vty-линиях?** -- *от 0 до 4 и от 5 до 15*

* c) Для изучения загрузочной конфигурации (startup configuration), воспользуемся командой:

```
Switch#show startup-config
```

в результате получим сообщение "startup-config is not present" - что означает, загрузочная информация еще не представлена (не сохранена)

* d) Изучим характеристики SVI для VLAN 1
```
Switch#show running-config
interface Vlan1 
 no ip address
 shutdown
```
ip-адрес не назначен, порт отключен

* e) Изучим IP-свойства интерфейса SVI сети VLAN 1.

```
Switch#show ip interface brief
  Interface              IP-Address      OK? Method Status                Protocol
  Vlan1                  unassigned      YES manual administratively down down
```

Видим тоже самое что и в предыдущем пункте. ip-адрес не назначен, порт отключен

* f) Подсоединим кабель Ethernet компьютеру PC-A к порту 6 на коммутаторе и изучите IP-свойства интерфейса SVI сети VLAN 1.

```
Switch#show ip interface brief
  Interface              IP-Address      OK? Method Status                Protocol
  FastEthernet0/6        unassigned      YES manual up                    up 
```

Мы видим активный порт 6, но ip-адрес не получен.

* g) Изучим сведения о версии ОС Cisco IOS на коммутаторе.
```
Switch#show version
  Switch Ports Model              SW Version            SW Image
------ ----- -----              ----------            ----------
*    1 26    WS-C2960-24TT-L    15.0(2)SE4            C2960-LANBASEK9-M
```

Используется ОС CISCO IOS версия 15.0(2)SE4. Файл образа системы называется: c2960-lanbasek9-mz.150-2.SE4.bin.

* h) Изучим свойства по умолчанию интерфейса FastEthernet, который используется компьютером PC-A.
```
Switch# show interface f0/6
 FastEthernet0/6 is up, line protocol is up (connected)
  Hardware is Lance, address is 00e0.8f41.6e06 (bia 00e0.8f41.6e06)
 BW 100000 Kbit, DLY 100 usec,
     reliability 255/255, txload 1/255, rxload 1/255
```

Интерфейс включен, сетевой кабель подключен. Для включения интерфейса используется команда "no shutdown".

* i)	Изучим флеш-память.

Выполним одну из команд:
```
Switch# show flash 
Switch# dir flash: 
Directory of flash:/

    1  -rw-     4670455           <no date>  2960-lanbasek9-mz.150-2.SE4.bin
```

Образу Cisco IOS присвоено имя: 2960-lanbasek9-mz.150-2.SE4.bin

### Часть 2. Настройка базовых параметров сетевых устройств

Шаг 1. Настроить базовые параметры коммутатора.

* a) В режиме глобальной конфигурации скопируем следующие базовые параметры конфигурации:
```
Switch#configure terminal
Enter configuration commands, one per line.  End with CNTL/Z.
Switch(config)#no ip domain-lookup
Switch(config)#hostname S1
S1(config)#service password-encryption
S1(config)#enable secret class
S1(config)#banner motd #
Enter TEXT message.  End with the character '#'.
Unauthorized access is strictly prohibited. #
```

* b) Назначим IP-адрес интерфейсу SVI на коммутаторе:
```  
S1#configure terminal
Enter configuration commands, one per line.  End with CNTL/Z.
S1(config)#interface vlan 1
S1(config-if)#ip address 192.168.1.2 255.255.255.0
S1(config-if)#no shutdown
S1(config-if)#
%LINK-3-UPDOWN: Interface Vlan1, changed state to down
%LINEPROTO-5-UPDOWN: Line protocol on Interface Vlan1, changed state to up
```

* с) Доступ через порт консоли также ограничим с помощью пароля:
```
S1(config)#line con 0
S1(config-line)#logging synchronous
S1(config-line)#password cisco
S1(config-line)#login
```

* d) Настроим каналы виртуального соединения для удаленного управления (vty)
```
S1(config)#line vty 0 4
S1(config-line)#password cisco
S1(config-line)#login
S1(config-line)#exit
S1(config)#service password-encryption 
```
Команда "login" нужна для передачи разрешения на авторизацию в пользовательском режиме

Шаг 2. Настроим IP-адрес на компьютере PC-A.



        
