# Лабораторная работа. Доступ к сетевым устройствам по протоколу SSH

## Топология:
![](Top.jpg)

## Таблица адресации:

|   Устройство   |   Интерфейс   |   IP-адрес        |   Маска подсети  |    Шлюз по умолчанию  |           
|  ------------  |  -----------  |  --------------   | -----------------|  -------------------- |
|   R1           |   G0/0/1      |   192.168.1.1     |  255.255.255.0   |  --                   |
|   S1           |   VLAN 1      |   192.168.1.11    |  255.255.255.0   |  192.168.1.1          |
|   PC-A         |   NIC         |   192.168.1.3     |  255.255.255.0   |  192.168.1.1          |

## Задачи:

### Часть 1. Настройка основных параметров устройства.

### Часть 2. Настройка маршрутизатора для доступа по протоколу SSH.

### Часть 3. Настройка коммутатора для доступа по протоколу SSH.

### Часть 4. SSH через интерфейс командной строки (CLI) коммутатора.

## Решение:

### Часть 1. Настройка основных параметров устройства.

Шаг 1. Создадим сеть согласно топологии в CISCO Packe Tracert.

Шаг 2. Выполниv инициализацию и перезагрузку маршрутизатора и коммутатора.

Шаг 3. Настроим маршрутизатор.

*  a) Подключимся к маршрутизатору с помощью консоли и активируем привилегированный режим EXEC.
  
*  b)	Войдем в режим конфигурации.

*  c)	Отключим поиск DNS, чтобы предотвратить попытки маршрутизатора неверно преобразовывать введенные команды таким образом, как будто они являются именами узлов.

*  d)	Назначим class в качестве зашифрованного пароля привилегированного режима EXEC.

*  e)	Назначим cisco в качестве пароля консоли и включите вход в систему по паролю.

*  f)	Назначим cisco в качестве пароля VTY и включите вход в систему по паролю.

*  g)	Зашифруем открытые пароли.

*  h)	Создадим баннер, который предупреждает о запрете несанкционированного доступа.

*  i)	Настроим и активируем на маршрутизаторе интерфейс G0/0/1, используя информацию, приведенную в таблице адресации.

*  j)	Сохраним текущую конфигурацию в файл загрузочной конфигурации.



```
Router>en
Router#conf t
Router(config)#no ip domain-lookup 
Router(config)#enable secret class
Router(config)#line con 0
Router(config-line)#password cisco
Router(config-line)#login
Router(config-line)#line vty 0 4
Router(config-line)#password cisco
Router(config-line)#login
Router(config-line)#exit
Router(config)#service password-encryption 
Router(config)#banner motd #
Enter TEXT message.  End with the character '#'.
Unauthorized access is strictly prohibited.#
Router(config)#int g0/0/1
Router(config-if)#ip address 192.168.1.1 255.255.255.0
Router(config-if)#no sh

Router(config-if)#
%LINK-5-CHANGED: Interface GigabitEthernet0/0/1, changed state to up

%LINEPROTO-5-UPDOWN: Line protocol on Interface GigabitEthernet0/0/1, changed state to up
Router(config-if)#^Z
Router#
%SYS-5-CONFIG_I: Configured from console by console

Router#copy run sta
Destination filename [startup-config]? 
Building configuration...
[OK]
Router#
```

Шаг 4. Настроим компьютер PC-A.

*  a)	Настроим для PC-A IP-адрес и маску подсети.

*  b)	Настроим для PC-A шлюз по умолчанию.

Шаг 5. Проверим подключение к сети.

Отправим с PC-A команду Ping на маршрутизатор R1.

```
C:\>ping 192.168.1.1

Pinging 192.168.1.1 with 32 bytes of data:

Reply from 192.168.1.1: bytes=32 time<1ms TTL=255
Reply from 192.168.1.1: bytes=32 time<1ms TTL=255
Reply from 192.168.1.1: bytes=32 time<1ms TTL=255
Reply from 192.168.1.1: bytes=32 time<1ms TTL=255

Ping statistics for 192.168.1.1:
    Packets: Sent = 4, Received = 4, Lost = 0 (0% loss),
Approximate round trip times in milli-seconds:
    Minimum = 0ms, Maximum = 0ms, Average = 0ms
```

### Часть 2. Настройка маршрутизатора для доступа по протоколу SSH.

Шаг 1. Настроим аутентификацию устройств.

*  a)	Зададим имя устройства.

*  b)	Задайте домен для устройства.

```
Router#
Router#conf t
Router(config)#hostname R1
R1(config)#ip domain-name test.com
```

Шаг 2. Создади ключ шифрования с указанием его длины.

```
R1(config)#crypto key gen rsa
The name for the keys will be: R1.test.com
Choose the size of the key modulus in the range of 360 to 4096 for your
  General Purpose Keys. Choosing a key modulus greater than 512 may take
  a few minutes.

How many bits in the modulus [512]: 1024
% Generating 1024 bit RSA keys, keys will be non-exportable...[OK]
```

Для повышения безопасности, обязательно включаем версию SSH 2.0.

```
ip ssh version 2
```

Шаг 3. Создадим имя пользователя в локальной базе учетных записей.

```
R1(config)#username admin secret Adm1nP@55
```

Шаг 4. Активируем протокол SSH на линиях VTY.

* a)  Активируем протокол SSH на входящих линиях VTY с помощью команды transport input.

* b)	Изменим способ входа в систему таким образом, чтобы использовалась проверка пользователей по локальной базе учетных записей.

```
R1(config-line)#transport input ssh
R1(config-line)#login local
```

Шаг 5. Сохраним текущую конфигурацию в файл загрузочной конфигурации.

```
R1#copy run sta
Destination filename [startup-config]? 
Building configuration...
[OK]
```

Шаг 6. Установим соединение с маршрутизатором по протоколу SSH.

* a)	Запустим Командную строку с PC-A.

* b)	Установим SSH-подключение к R1. 

```
C:\>ssh -l admin 192.168.1.1

Password: 


Unauthorized access is strictly prohibited.

R1>
```



