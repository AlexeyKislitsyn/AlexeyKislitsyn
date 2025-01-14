# Настройка NAT для IPv4

### Топология

![](1.png)

### Таблица адресации

| Устройство  | Интерфейс    | IP  -адрес         | Маска подсети   |
|-------------|--------------|--------------------|-----------------|
| R1          | e 0/0        | 209.165.200.230    | 255.255.255.248 | 
| R1          | e 0/1        | 192.168.1.1        | 255.255.255.0   | 
| R2          | e 0/0        | 209.165.200.225    | 255.255.255.248 | 
| R2          | Lo 1         | 209.165.200.1      | 255.255.255.224 | 
| S1          | VLAN 1       | 192.168.1.11       | 255.255.255.0   | 
| S2          | VLAN 1       | 192.168.1.12       | 255.255.255.0   | 
| PC-A        | NIC          | 192.168.1.2        | 255.255.255.0   | 
| PC-B        | NIC          | 192.168.1.3        | 255.255.255.0   | 

Интернет-провайдер выделил компании общедоступное пространство IP-адресов 209.165.200.224/29. Эта сеть используется для обращения к каналу между маршрутизатором ISP (R2) и шлюзом компании (R1). Первый адрес (209.165.200.225) назначается интерфейсу e 0/0 на R2, а последний адрес (209.165.200.230) назначается интерфейсу e 0/0 на R1. Остальные адреса (209.165.200.226-209.165.200.229) будут использоваться для предоставления доступа в Интернет хостам компании. Маршрут по умолчанию используется от R1 до R2. Подключение интернет-провайдера к Интернету смоделировано loopback-адресом на маршрутизаторе интернет-провайдера.

#### Задачи

1. Создание сети и настройка основных параметров устройства
2. Настройка и проверка NAT для IPv4
3. Настройка и проверка PAT для IPv4
4. Настройка и проверка статического NAT для IPv4

---

### Шаг 1. Создание сети и настройка основных параметров устройства

* Создана топология сети в EVE-NG.
* Выполним настройку интерфейсов активного оборудования согласно талицы ip адресации.

```
S1#sh ip int brief 
Interface              IP-Address      OK? Method Status                Protocol
Ethernet0/0            unassigned      YES unset  up                    up      
Ethernet0/1            unassigned      YES unset  up                    up      
Ethernet0/2            unassigned      YES unset  up                    up      
Ethernet0/3            unassigned      YES unset  up                    up      
Vlan1                  192.168.1.11    YES manual up                    up      
S1#
S1#sh ip redirects 
Default gateway is 192.168.1.1


S2#sh ip int brief 
Interface              IP-Address      OK? Method Status                Protocol
Ethernet0/0            unassigned      YES unset  up                    up      
Ethernet0/1            unassigned      YES unset  up                    up      
Ethernet0/2            unassigned      YES unset  up                    up      
Ethernet0/3            unassigned      YES unset  up                    up      
Vlan1                  192.168.1.12    YES manual up                    up      
S2#
S2#sh ip redirects 
Default gateway is 192.168.1.1

R1#sh ip int brief 
Interface                  IP-Address      OK? Method Status                Protocol
Ethernet0/0                209.165.200.230 YES manual up                    up      
Ethernet0/1                192.168.1.1     YES manual up                    up      
Ethernet0/2                unassigned      YES unset  administratively down down    
Ethernet0/3                unassigned      YES unset  administratively down down    
R1#

R1#sh ip route static 
S*    0.0.0.0/0 [1/0] via 209.165.200.225

R2#sh ip int brief 
Interface                  IP-Address      OK? Method Status                Protocol
Ethernet0/0                209.165.200.225 YES manual up                    up      
Ethernet0/1                unassigned      YES unset  administratively down down    
Ethernet0/2                unassigned      YES unset  administratively down down    
Ethernet0/3                unassigned      YES unset  administratively down down    
Loopback1                  209.165.200.1   YES manual up                    up      
R2#


```













SW2#sh ip int brief 
Interface              IP-Address      OK? Method Status                Protocol
Ethernet0/0            unassigned      YES unset  up                    up      
Ethernet0/1            unassigned      YES unset  up                    up      
Ethernet0/2            unassigned      YES unset  up                    up      
Ethernet0/3            unassigned      YES unset  administratively down down    
Vlan20                 10.20.0.3       YES manual down                  down    
SW2#

SW2#sh vlan

VLAN Name                             Status    Ports
---- -------------------------------- --------- -------------------------------
1    default                          active    Et0/0, Et0/1
20   mng                              active    
30   operations                       active    
40   sales                            active    Et0/2
999  PL                               active    Et0/3

SW2#sh ip redirects 
Default gateway is 10.20.0.1
```
* Настроим на коммутаторах магистральные каналы, разрешим vlan 20,30,40,1000, и назнаим native vlan:

```
SW1(config)# int r e 0/0-1
SW1(config-if-range)#switchport trunk encapsulation dot1q 
SW1(config-if-range)#switchport mode trunk 
SW1(config-if-range)#no shutdown 
SW1(config-if-range)#switchport trunk allowed vlan 20,30,40,1000
SW1(config-if-range)#switchport trunk native vlan 1000

SW1#sh interfaces trunk 

Port        Mode             Encapsulation  Status        Native vlan
Et0/0       on               802.1q         trunking      1000
Et0/1       on               802.1q         trunking      1000

Port        Vlans allowed on trunk
Et0/0       20,30,40,1000
Et0/1       20,30,40,1000

Port        Vlans allowed and active in management domain
Et0/0       20,30,40,1000
Et0/1       20,30,40,1000

Port        Vlans in spanning tree forwarding state and not pruned
Et0/0       20,30,40,1000
Et0/1       20,30,40,1000
SW1#

SW2#sh interfaces trunk 

Port        Mode             Encapsulation  Status        Native vlan
Et0/0       on               802.1q         trunking      1000
Et0/1       on               802.1q         trunking      1000

Port        Vlans allowed on trunk
Et0/0       20,30,40,1000
Et0/1       20,30,40,1000

Port        Vlans allowed and active in management domain
Et0/0       20,30,40,1000
Et0/1       20,30,40,1000

Port        Vlans in spanning tree forwarding state and not pruned
Et0/0       20,30,40,1000
Et0/1       20,30,40,1000
SW2#
```

* Настроим саб интерфейсы на R1:

