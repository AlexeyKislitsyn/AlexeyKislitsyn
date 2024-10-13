# Лабораторная работа. Доступ к сетевым устройствам по протоколу SSH 

### Топология

![](lab5.png)

### Таблица адресации

| Устройство  | Интерфейс | IP  -адрес          | Маска подсети  | Шлюз по умолчанию |
|-------------|-----------|---------------------|----------------|-------------------|
| R1          | G0/0/0    | 192.168.1.1         | 255.255.255.0  | -                 | 
| S1          | VLAN 1    | 192.168.1.11        | 255.255.255.0  | 192.168.1.1       | 
| PC-A        | NIC       | 192.168.1.3         | 255.255.255.0  | 192.168.1.1       | 

### Часть 1. Настройка основных параметров устройств

В CPT создана лаборатория:

![](1.png)

### Шаг 1.  Настройка маршрутизатора.

```
Router#write erase
Router#reload
Router#conf t
Router(config)#hostname R1
R1(config)#no ip domain-lookup 
R1(config)#enable secret class
R1(config)#line console 0
R1(config-line)#password cisco
R1(config-line)#login
R1(config)#service password-encryption 
R1(config)#banner motd "Attention"
R1(config)#int g 0/0/1
R1(config-if)#ip address 192.168.1.1 255.255.255.0
R1(config-if)#no shutdown 
R1#wr
```

### Шаг 2. Настройка компьютера PC-A.

![](pc.png)


### Шаг 3. Проверка подключение к сети..
```
C:\>ping 192.168.1.1

Pinging 192.168.1.1 with 32 bytes of data:

Reply from 192.168.1.1: bytes=32 time=8ms TTL=255
Reply from 192.168.1.1: bytes=32 time<1ms TTL=255
Reply from 192.168.1.1: bytes=32 time<1ms TTL=255
Reply from 192.168.1.1: bytes=32 time<1ms TTL=255

Ping statistics for 192.168.1.1:
    Packets: Sent = 4, Received = 4, Lost = 0 (0% loss),
Approximate round trip times in milli-seconds:
    Minimum = 0ms, Maximum = 8ms, Average = 2ms

C:\>
```

### Часть 2. Настройка маршрутизатора для доступа по протоколу SSH

### Шаг 1. Настройка аутентификацию устройств.

При генерации ключа шифрования в качестве его части используются имя устройства и домен. Имя устройства уже задано, т.о необходимо указать домен:

```
R1(config)#ip domain-name alex.com
```

###  Шаг 2. Создание ключа шифрования с указанием его длины.

```
R1(config)#crypto key generate rsa 
The name for the keys will be: R1.alex.com
Choose the size of the key modulus in the range of 360 to 4096 for your
  General Purpose Keys. Choosing a key modulus greater than 512 may take
  a few minutes.

How many bits in the modulus [512]: 2048
% Generating 2048 bit RSA keys, keys will be non-exportable...[OK]

R1(config)#
```

так же укажем ssh ver 2

```
R1(config)#ip ssh version 2
```

### Шаг 3. Создайние имя пользователя в локальной базе учетных записей.

```
R1(config)#username admin privilege 15 secret Adm1nP@55
```

### Шаг 4. Активация протокола SSH на линиях VTY.

Активация протоколов Telnet и SSH на входящих линиях VTY с помощью команды transport input.
Используется проверка пользователей по локальной базе учетных записей.

```
R1(config)#line vty 0 4
R1(config-line)#transport input ssh
R1(config-line)#login local


R1(config)#line vty 5
R1(config-line)#transport input telnet
R1(config-line)#login local
```

### Шаг 5. Установление соединение с маршрутизатором по протоколу SSH.

Подключимся к R1 по SSH из командной строки PC:

```
C:\>ssh -l admin 192.168.1.1

Password: 

Attention

R1#
```

Подключение выполнено усешно. 

### Часть 3. Настройка коммутатора для доступа по протоколу SSH

Данные настройки будут аналогичны настрокам R1.

```
Switch#write erase
Switch#conf t
Switch(config)#no ip domain-lookup 
Switch(config)#enable secret cisco
Switch(config)#service password-encryption 
Switch(config)#line con 0
Switch(config-line)#password cisco
Switch(config-line)#login
Switch(config-line)#exit
Switch(config)#int vlan 1
Switch(config-if)#ip address 192.168.1.11 255.255.255.0
Switch(config-if)#no shutdown  
Switch(config-if)#exit
S1(config)#ip default-gateway 192.168.1.1

SSH:

Switch(config)#hostname S1
S1(config)#ip domain name alex.com
S1(config)#crypto key generate rsa
The name for the keys will be: S1.alex.com
Choose the size of the key modulus in the range of 360 to 4096 for your
  General Purpose Keys. Choosing a key modulus greater than 512 may take
  a few minutes.

How many bits in the modulus [512]: 2048
% Generating 2048 bit RSA keys, keys will be non-exportable...[OK]

S1(config)#ip ssh ver 2
*Mar 1 0:4:36.657: %SSH-5-ENABLED: SSH 1.99 has been enabled
S1(config)#user admin privilege 15 secret Adm1nP@55
S1(config)#line vty 0 4
S1(config-line)#transport input ssh 
S1(config-line)#login local 	
S1(config-line)#exit
S1(config)#line vty 5
S1(config-line)#login local
S1(config-line)#transport input telnet 
S1(config)#do wr
```

Проверим подключение по SSH c PC на S1:

```
C:\>ssh -l admin 192.168.1.11

Password: 

S1#
```

Успешно.


### Часть 4. Настройка протокола SSH с использованием интерфейса командной строки (CLI) коммутатора

Посмотрим варианты параметров для команды ssh:

```
S1#ssh ?
  -l  Log in using this user name
  -v  Specify SSH Protocol Version
S1#
```

Установим с коммутатора S1 соединение с маршрутизатором R1 по протоколу SSH.

```
S1#ssh -l admin 192.168.1.1

Password: 

Attention

R1#
```

Чтобы вернуться к коммутатору S1, не закрывая сеанс SSH с маршрутизатором R1, используем комбинацию клавиш Ctrl+Shift+6. Отпустим клавиши Ctrl+Shift+6 и нажмимаем x. Отображается приглашение привилегированного режима EXEC коммутатора.

R1#

S1#

Чтобы вернуться к сеансу SSH на R1, нажмимаем клавишу Enter в пустой строке интерфейса командной строки. Чтобы увидеть окно командной строки маршрутизатора, нажимаем клавишу Enter еще раз.

S1#

Resuming connection 1 to 192.168.1.1 ... 

R1#

Чтобы завершить сеанс SSH на маршрутизаторе R1, вводим в командной строке маршрутизатора команду exit.

R1# exit

Connection to 192.168.1.1 closed by foreign host

S1#

При использовании интерфейса командной строки поддерживаются 1 и 2 версии протокола SSH.

Для предоставления доступа к сетевому устройству нескольким пользователям, у каждого из которых есть собственное имя пользователя можно использовать команду username. Так-же можно сконфигурировать права на выполнение тех или иных команд для каждого пользователя.


