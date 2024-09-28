# Лабораторная работа. Настройка IPv6-адресов на сетевых устройствах 

### Топология

![](lab4.png)

### Таблица адресации

| Устройство  | Интерфейс | IPv6-адрес          | Link local IPv6-адрес  | Длина префикса    | Шлюз по умолчанию |
|-------------|-----------|---------------------|------------------------|-------------------|-------------------|
| R1          | G0/0/0    | 2001:db8:acad:a::1  | fe80::1                | 64                | —                 |
| R1          | G0/0/1    | 2001:db8:acad:1::1  | fe80::1                | 64                | —                 |
| S1          | VLAN 1    | 2001:db8:acad:1::b  | fe80::b                | 64                | —                 |
| PC-A        | NIC       | 2001:db8:acad:1::3  | SLACC                  | 64                | fe80::1           |
| PC-B        | NIC       | 2001:db8:acad:a::3  | SLACC                  | 64                | fe80::1           |


### Часть 1. Настройка топологии и конфигурация основных параметров маршрутизатора и коммутатора

В CPT создана лаоратория:
![](1.png)

### Шаг 1.  Настройте маршрутизатор.

Назначьте имя хоста и настройте основные параметры устройства.

```
Router#write erase 
Router#conf t
Router(config)#hostname R1
R1(config)#enable secret cisco
R1(config)#line vty 0 4
R1(config-line)#password cisco
R1(config-line)#transport input telnet 
R1(config-line)#login 
R1(config)#line console 0
R1(config-line)#password cisco
R1(config-line)#login
R1(config)#no ip domain-lookup 
R1(config)#service password-encryption 
R1#wr
```

### Шаг 2. Настройте коммутатор.

Назначьте имя хоста и настройте основные параметры устройства.

```
Switch#write erase
Switch#conf t
Switch(config)#hostname S1
S1(config)#enable secret cisco
S1(config)#line vty 0 4
S1(config-line)#password cisco
S1(config-line)#transport input telnet
S1(config-line)#login
S1(config)#line con 0
S1(config-line)#password cisco
S1(config-line)#login
S1(config)#service password-encryption 
S1(config)#no ip domain-lookup 
S1(config)#do wr
```

### Часть 2. Ручная настройка IPv6-адресов

### Шаг 1. Назначьте IPv6-адреса интерфейсам Ethernet на R1.

Выполним команды:

```
R1#conf t
R1(config)#int g 0/0/0
R1(config-if)#ipv6 add
R1(config-if)#ipv6 address 2001:db8:acad:a::1/64 
R1(config-if)#no sh
%LINK-5-CHANGED: Interface GigabitEthernet0/0/0, changed state to up
%LINEPROTO-5-UPDOWN: Line protocol on Interface GigabitEthernet0/0/0, changed state to up
R1(config-if)#exit
R1(config)#int g 0/0/1
R1(config-if)#ipv6 address 2001:db8:acad:1::1/64 
R1(config-if)#no sh
%LINK-5-CHANGED: Interface GigabitEthernet0/0/1, changed state to up
%LINEPROTO-5-UPDOWN: Line protocol on Interface GigabitEthernet0/0/1, changed state to up
R1(config-if)#
```

Проверим корректность введенных данных:

```
R1#sh ipv6 int br
GigabitEthernet0/0/0       [up/up]
    FE80::201:97FF:FEE6:3A01
    2001:DB8:ACAD:A::1
GigabitEthernet0/0/1       [up/up]
    FE80::201:97FF:FEE6:3A02
    2001:DB8:ACAD:1::1
Vlan1                      [administratively down/down]
    unassigned
R1#
```

Так же, автоматиески создался link-local адрес, начинается на FE80. Отображаемый локальный адрес канала основан на адресации EUI-64, которая автоматически использует MAC-адрес интерфейса для создания 128-битного локального IPv6-адреса канала.

Назначим вручную  link-local адреса на интерфесах R1. Пакеты с локальным адресом канала никогда не выходят за пределы локальной сети, а значит, для обоих интерфейсов можно указывать один и тот же локальный адрес канала.

```
R1#conf t
R1(config)#int g 0/0/0
R1(config-if)#ipv6 address fe80::1 link-local 
R1(config-if)#ex
R1(config)#int g 0/0/1
R1(config-if)#ipv6 address fe80::1 link-local 
R1(config-if)#
```

Убедимся, что адрес link-local изменился:

```
R1#sh ipv6 int brief 
GigabitEthernet0/0/0       [up/up]
    FE80::1
    2001:DB8:ACAD:A::1
GigabitEthernet0/0/1       [up/up]
    FE80::1
    2001:DB8:ACAD:1::1
Vlan1                      [administratively down/down]
    unassigned
R1#
```

Так же, можно увидеть, какие группы многоадресной рассылки назначены интерфейсу. Рассмотрим на примере G0/0/0:

```
R1# sh ipv6 int g 0/0/0
GigabitEthernet0/0/0 is up, line protocol is up
  IPv6 is enabled, link-local address is FE80::1
  No Virtual link-local address(es):
  Global unicast address(es):
    2001:DB8:ACAD:A::1, subnet is 2001:DB8:ACAD:A::/64
  Joined group address(es):
    FF02::1
    FF02::1:FF00:1
  MTU is 1500 bytes
  ICMP error messages limited to one every 100 milliseconds
  ICMP redirects are enabled
  ICMP unreachables are sent
  ND DAD is enabled, number of DAD attempts: 1
  ND reachable time is 30000 milliseconds
R1#
```
Joined group address(es) - группы многоадрессной рассылки: FF02::1 FF02::1:FF00:1

### Шаг 2. Активируйте IPv6-маршрутизацию на R1.

Часть 3. Проверка сквозного соединения


