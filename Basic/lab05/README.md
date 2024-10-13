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
