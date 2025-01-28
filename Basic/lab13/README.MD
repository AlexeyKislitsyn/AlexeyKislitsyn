
# Настройка протоколов CDP, LLDP и NTP

### Топология

![](11.png)

### Таблица адресации

| Устройство  | Интерфейс    | IP  -адрес         | Маска подсети  | Шлюз по умолчанию |
|-------------|--------------|--------------------|----------------|-------------------|
| R1          | Loopback 0   | 172.16.1.1         | 255.255.255.0  | -                 | 
| R1          | e 0/0     | 10.22.0.1          | 255.255.255.0  | -                 | 
| R1          | SVI VLAN 1   | 10.22.0.2          | 255.255.255.0  | 10.22.0.1         | 
| R1          | SVI VLAN 1   | 10.22.0.3          | 255.255.255.0  | 10.22.0.1         | 


### Задачи
Создание сети и настройка основных параметров устройства
Обнаружение сетевых ресурсов с помощью протокола CDP
Обнаружение сетевых ресурсов с помощью протокола LLDP
Настройка и проверка NTP

---

### Шаг 1. Создание сети и настройка основных параметров устройства

* Создана топология сети в EVE-NG.
* Выполнена базовая настройка маршрутизаторов и коммутаторов по шаблону.
* Выполним настройку VLAN и SVI на коммутаторах согласно таблице VLAN и таблице адресации:

```
SW1(config)#vlan 20
SW1(config-vlan)#name mng
SW1(config-vlan)#vlan 30
SW1(config-vlan)#name operations
SW1(config-vlan)#vlan 40
SW1(config-vlan)#name SALES
SW1(config-vlan)#vlan 999   
SW1(config-vlan)#name PL
SW1(config-vlan)#vlan 1000
SW1(config-vlan)#name native
SW1(config-vlan)#int vlan 20
SW1(config-if)#ip address 10.20.0.2 255.255.255.0
SW1(config)#ip default-gateway 10.20.0.1
SW1(config)#int e 0/2
SW1(config-if)#switchport mode access 
SW1(config-if)#switchport access vlan 30
SW1(config-if)#no shutdown 
SW1(config)#int e 0/3
SW1(config-if)#switchport mode access 
SW1(config-if)#switchport access vlan 999
SW1(config-if)#shutdown 



SW1#sh ip int brief 
Interface              IP-Address      OK? Method Status                Protocol
Ethernet0/0            unassigned      YES unset  up                    up      
Ethernet0/1            unassigned      YES unset  up                    up      
Ethernet0/2            unassigned      YES unset  up                    up      
Ethernet0/3            unassigned      YES unset  administratively down down    
Vlan20                 10.20.0.2       YES manual down                  down 

SW1#sh vlan

VLAN Name                             Status    Ports
---- -------------------------------- --------- -------------------------------
1    default                          active    Et0/0, Et0/1
20   mng                              active    
30   operations                       active    Et0/2
40   SALES                            active    
999  PL                               active    Et0/3

SW1#sh ip redirects 
Default gateway is 10.20.0.1
```

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

```
R1(config)#int loop 0
R1(config-if)#ip address 172.16.1.1 255.255.255.0
R1(config-if)#int e 0/0.20
R1(config-subif)#description mng 
R1(config-subif)#encapsulation dot1Q 20
R1(config-subif)#ip address 10.20.0.1 255.255.255.0
R1(config-subif)#int e 0/0.30                       
R1(config-subif)#description operations            
R1(config-subif)#encapsulation dot1Q 30            
R1(config-subif)#ip address 10.30.0.1 255.255.255.0
R1(config-subif)#int e 0/0.40                      
R1(config-subif)#description sales                 
R1(config-subif)#encapsulation dot1Q 40            
R1(config-subif)#ip address 10.40.0.1 255.255.255.0
R1(config-subif)#int e 0/0.1000                    
R1(config-subif)#description native                
R1(config-subif)#encapsulation dot1Q 1000 
R1(config)#int e 0/0
R1(config-if)#no sh

R1#sh ip int brief 
Interface                  IP-Address      OK? Method Status                Protocol
Ethernet0/0                unassigned      YES unset  up                    up      
Ethernet0/0.20             10.20.0.1       YES manual up                    up      
Ethernet0/0.30             10.30.0.1       YES manual up                    up      
Ethernet0/0.40             10.40.0.1       YES manual up                    up      
Ethernet0/0.1000           unassigned      YES unset  up                    up      
Ethernet0/1                unassigned      YES unset  administratively down down    
Ethernet0/2                unassigned      YES unset  administratively down down    
Ethernet0/3                unassigned      YES unset  administratively down down    
Loopback0                  172.16.1.1      YES manual up                    up      
R1#
```

* Настроим саб интерфейсы на R2, и опишем маршрут по умолчанию (на R1):

```
