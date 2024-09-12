# Лабораторная работа. Базовая настройка коммутатора.

### Топология.
![](1.png)

### Таблица адресации
```
 S1  -  VLAN1  -  192.168.1.2/24
PC-A -   NIC   -  192.168..10/24
```
# Часть 1. Создание сети и проверка настроек коммутатора по умолчанию.
### Шаг 1  Создайте сеть согласно топологии.
  Выполнено создание сети согласно топпологии. Установлено консольное подключение к коммутатору с помощью программы эмуляции терминала.
  
  Консольное подключение используется при начальной настройке коммутатора не имеющего сетевого доступа.Только после начальной настройки коммутатора мы можем использовать удаленное подключение Telnet, SSH. 
### Шаг 2  Проверьте настройки коммутатора по умолчанию. 
a. Просмотр show runnin-cinfig. Для полноты сбросим к заводским настройкам, если есть vlan.dat удалим его и перезагрузим коммутатор. Используем команды:
```
Switch# show flash
Switch# delete vlan.dat
Switch#erase startup-config. 
Switch#reload
```
b. Изучите текущий файл running configuration.
  
Имеем: 24 интерфейса FastEthernet, 2 интерфейса GigabitEthernet;
   
Диапазон значений, отображаемых в vty-линиях: 0-4, 5-15.
   
с. Изучите файл загрузочной конфигурации startup configuration

```
Switch#sh startup-config
startup-config is not present
```

Данное сообщение говорит о том, что еще не производилась запись конфига в  startup-config.
   
d. Изучите характеристики SVI для VLAN 1.
```
Switch#sh int vlan 1.
```

ip для VLAN 1 еще не назначен.

mac vlan 1 :  0010.11e9.703b.

интерфейс отключен:  Vlan1 is administratively down, line protocol is down.

e. Изучите IP-свойства интерфейса SVI сети VLAN 1.

```
Switch#sh ip int vlan 1
```

g. Изучите сведения о версии ОС Cisco IOS на коммутаторе.

```
Switch#sh ver
Switch#sh flash:
```

Cisco IOS  :  C2960 Software (C2960-LANBASEK9-M), Version 15.0(2)SE4
    
файл образа :  System image file is "flash:c2960-lanbasek9-mz.150-2.SE4.bin"

h. Изучите свойства по умолчанию интерфейса FastEthernet, который используется компьютером PC-A.
    
```
Switch#sh int fast 0/6
```
    
по умолчанию интерфейс в down : FastEthernet0/6 is down, line protocol is down (disabled)

включить интерфейс: Switch(config-if)# no shutdown 

mac 0001.4333.4506

Full-duplex, 100Mb/s

i. Изучите флеш-память.
    
```
Switch#flash:
Switch#sh dir
```
    
файл образа :  System image file is "flash:c2960-lanbasek9-mz.150-2.SE4.bin"

# Часть 2. Настройка базовых параметров сетевых устройств
### Шаг 1. Настройте базовые параметры коммутатора

Выполнена базовая настройка коммутатора согласно методички. Команда login запрашивает пароль при подключении.


### Шаг 2. Настройте IP-адрес на компьютере PC-A.

Выполнена настройка ip адресации на PC-A

# Часть 3. Проверка сетевых подключений
### Шаг 1. Отобразите конфигурацию коммутатора.

```
S1#sh run
Building configuration...

Current configuration : 1392 bytes
!
version 15.0
no service timestamps log datetime msec
no service timestamps debug datetime msec
service password-encryption
!
hostname S1
!
enable secret 5 $1$mERr$9cTjUIEqNGurQiFU.ZeCi1
!
!
!
no ip domain-lookup
!
!
!
spanning-tree mode pvst
spanning-tree extend system-id
!
interface FastEthernet0/1
!
interface FastEthernet0/2
!
interface FastEthernet0/3
!
interface FastEthernet0/4
!
interface FastEthernet0/5
!
interface FastEthernet0/6
!
interface FastEthernet0/7
!
interface FastEthernet0/8
!
interface FastEthernet0/9
!
interface FastEthernet0/10
!
interface FastEthernet0/11
!
interface FastEthernet0/12
!
interface FastEthernet0/13
!
interface FastEthernet0/14
!
interface FastEthernet0/15
!
interface FastEthernet0/16
!
interface FastEthernet0/17
!
interface FastEthernet0/18
!
interface FastEthernet0/19
!
interface FastEthernet0/20
!
interface FastEthernet0/21
!
interface FastEthernet0/22
!
interface FastEthernet0/23
!
interface FastEthernet0/24
!
interface GigabitEthernet0/1
!
interface GigabitEthernet0/2
!
interface Vlan1
 ip address 192.168.1.2 255.255.255.0
!
ip default-gateway 192.68.1.1
!
banner motd ^C
Unauthorized access is strictly prohibited. ^C
!
!
!
line con 0
 password 7 0822455D0A16
 logging synchronous
 login
!
line vty 0 4
 password 7 0822455D0A16
 login
 transport input telnet
line vty 5 15
 password 7 0822455D0A16
 login
!
!
!
!
end
S1#
```
Выполнена проверка параметров Vlan. Полоса пробускания BW 100000 Kbit.

# Шаг 2. Протестируйте сквозное соединение, отправив эхо-запрос.

```
C:\>ping 192.168.1.2

Pinging 192.168.1.2 with 32 bytes of data:

Reply from 192.168.1.2: bytes=32 time<1ms TTL=255
Reply from 192.168.1.2: bytes=32 time<1ms TTL=255
Reply from 192.168.1.2: bytes=32 time<1ms TTL=255
Reply from 192.168.1.2: bytes=32 time=11ms TTL=255

Ping statistics for 192.168.1.2:
    Packets: Sent = 4, Received = 4, Lost = 0 (0% loss),
Approximate round trip times in milli-seconds:
    Minimum = 0ms, Maximum = 11ms, Average = 2ms
```

# Шаг 3. Проверьте удаленное управление коммутатором S1

С помощью Telnet выполнено удаленное подключение к коммутатору по адресу 192.168.1.2. Успешно.

Для удаленного доступа необходима настройка пароля на vty линиях.

Чтобы пароли не отправлялись в незашифрованном виде необходима настройка SSH соединения.
