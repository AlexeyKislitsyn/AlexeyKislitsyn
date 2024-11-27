
# Лабораторная работа - Конфигурация безопасности коммутатора 

### Топология

![](1.png)

### Таблица адресации

| Устройство  | Интерфейс   | IP  -адрес          | Маска подсети  | Шлюз по умолчанию |
|-------------|-------------|---------------------|----------------|-------------------|
| R1          | F0/0.10     | 192.168.10.1        | 255.255.255.0  | -                 | 
| R1          | F0/0.20     | 192.168.20.1        | 255.255.255.0  | -                 | 
| R1          | F0/0.333    | -                   | -              | -                 | 
| R1          | Loopback 0  | 10.10.1.1           | -              | -                 | 
| S1          | VLAN 10     | 192.168.10.201      | 255.255.255.0  | 192.168.10.1      | 
| S2          | VLAN 10     | 192.168.10.202      | 255.255.255.0  | 192.168.10.1      | 
|PC1          | NIC         | dhcp                | -              | -                 |
|PC2          | NIC         | dhcp                | -              | -                 |

### Таблица VLAN

| VLAN        |    Имя       | Назначенный интерфейс         | 
|-------------|--------------|-------------------------------|
| 10          | Управление   | S1: VLAN 10 , S2: VLAN 10     | 
| 20          | Students     | S1: E0/0, S2:E0/1             |  
| 333         | native       | -                             |
| 999         | Parking_Lot  | все не активные порты         |  


### Шаг 1. Настройка коммутаторов S1 и S2.

На коммутаторе S1 и S2 настроим: 1. vlan (пользовательский и управления), native vlan, svi 2. Настроим шлюз по умолчанию 3. Переведем магистральные порты в режим транка, настроим native vlan 4.На всех access портах оключим DTP 5. Неиспользуемые порты административно отключим и назначим их в 999 vlan 5. порты коммутатора куда подклюены ПК, назначим в пользовательский vlan 20.


S1(config)#no ip domain-lookup
S1(config)#vlan 10
S1(config-vlan)#name management
S1(config-vlan)#vlan 20            
S1(config-vlan)#name students
S1(config)#vlan 333
S1(config-vlan)#name native
S1(config-vlan)#vlan 999   
S1(config-vlan)#name parking_lot
S1(config)#int vlan 10       
S1(config-if)#ip address 192.168.10.201 255.255.255.0
S1(config-if)#no shutdown 
S1(config)#ip default-gateway 192.168.10.1
S1(config)#int g 3/2
S1(config)#description to_S1
S1(config-if)#switchport trunk encapsulation dot1q 
S1(config-if)#switchport mode trunk 
S1(config-if)#switchport trunk native vlan 333
S1(config-if)#switchport trunk allowed vlan 10,20,333
S1(config)#int g 0/0
S1(config)#description to_R1
S1(config-if)#switchport trunk encapsulation dot1q
S1(config-if)#switchport mode trunk 
S1(config-if)#switchport trunk native vlan 333
S1(config-if)#switchport trunk allowed vlan 10,20,333
S1(config)#int g 0/1
S1(config-if)#switchport mode access 
S1(config-if)#switchport access vlan 20
S1(config)#int range g 0/2-3,g 1/0-3,g 2/0-3,g 3/0-1
S1(config-if-range)#switchport mode access 
S1(config-if-range)#switchport access vlan 999
S1(config-if-range)#no negotiation auto 
S1(config-if-range)#shutdown 


S1#sh interfaces status 

Port      Name               Status       Vlan       Duplex  Speed Type 
Gi0/0     to_R1              connected    trunk        auto   auto unknown
Gi0/1                        connected    20           auto   auto unknown
Gi0/2                        disabled     999          auto   auto unknown
Gi0/3                        disabled     999          auto   auto unknown
Gi1/0                        disabled     999          auto   auto unknown
Gi1/1                        disabled     999          auto   auto unknown
Gi1/2                        disabled     999          auto   auto unknown
Gi1/3                        disabled     999          auto   auto unknown
Gi2/0                        disabled     999          auto   auto unknown
Gi2/1                        disabled     999          auto   auto unknown
Gi2/2                        disabled     999          auto   auto unknown
Gi2/3                        disabled     999          auto   auto unknown
Gi3/0                        disabled     999          auto   auto unknown
Gi3/1                        disabled     999          auto   auto unknown
Gi3/2     to_S2              connected    trunk        auto   auto unknown
S1# 


S1#sh int trunk 

Port        Mode             Encapsulation  Status        Native vlan
Gi0/0       on               802.1q         trunking      333
Gi3/2       on               802.1q         trunking      333

Port        Vlans allowed on trunk
Gi0/0       10,20,333
Gi3/2       10,20,333

Port        Vlans allowed and active in management domain
Gi0/0       10,20,333
Gi3/2       10,20,333

Port        Vlans in spanning tree forwarding state and not pruned
Gi0/0       10,20,333
Gi3/2       10,20,333
S1#

Аналогичная настройка для S2: 
исклюение

S2(config)#int vlan 10
S2(config-if)#ip address 192.168.10.202 255.255.255.0


### Шаг 2. Настройка маршрутизатора R1.

1. Настроим саб интерфейсы 2. Настроим dhcp пул для абонентов vlan 20 3. Укажем доверенный порт, за которым находится dhcp сервер 

R1(config)#ip dhcp excluded-address 192.168.20.1 
R1(config)#ip dhcp pool students
R1(dhcp-config)#network 192.168.20.0 255.255.255.0    
R1(dhcp-config)#default-router 192.168.20.1  
R1(dhcp-config)#domain-name test.ru
R1(config)#int f 0/0.10
R1(config-subif)#encapsulation dot1Q 10
R1(config-subif)#ip address 192.168.10.1 255.255.255.0
R1(config-subif)#int f 0/0.20                         
R1(config-subif)#encapsulation dot1Q 20               
R1(config-subif)#ip address 192.168.20.1 255.255.255.0
R1(config-subif)#ip dhcp relay information trusted 
R1(config-subif)#int f 0/0.333                                 
R1(config-subif)#encapsulation dot1Q 333 native 


R1#sh ip dhcp binding 
Bindings from all pools not associated with VRF:
IP address          Client-ID/	 	    Lease expiration        Type
		    Hardware address/
		    User name
192.168.20.2        0100.5079.6668.00       Mar 02 2002 01:31 AM    Automatic
192.168.20.3        0100.5079.6668.01       Mar 02 2002 01:31 AM    Automatic
R1#


### Шаг 3. Реализовать безопасность DHCP snooping.

Настроим DHCP snooping в VLAN 20 и укажем магистральный порт g 3/2 на S2 как доверенный.

S2(config)#ip dhcp snooping 
S2(config)#ip dhcp snooping vlan 20
S2(config)#int g 3/2                              
S2(config-if)#ip dhcp snooping trust 

Ограничим ненадежный порт g0/0 на S2 пятью DHCP-пакетами в секунду.

S2(config)#int  g0/0
S2(config-if)#ip dhcp snooping limit rate 5

Проверим DHCP Snooping на S2:

S2#sh ip dhcp snooping 
Switch DHCP snooping is enabled
Switch DHCP gleaning is disabled
DHCP snooping is configured on following VLANs:
20
DHCP snooping is operational on following VLANs:
20
DHCP snooping is configured on the following L3 Interfaces:

Insertion of option 82 is enabled
   circuit-id default format: vlan-mod-port
   remote-id: 0c3d.1fa0.0000 (MAC)
Option 82 on untrusted port is not allowed
Verification of hwaddr field is enabled
Verification of giaddr field is enabled
DHCP snooping trust/rate is configured on the following Interfaces:

Interface                  Trusted    Allow option    Rate limit (pps)
-----------------------    -------    ------------    ----------------   
GigabitEthernet0/0         no         no              5         
  Custom circuit-ids:
GigabitEthernet3/2         yes        yes             unlimited
  Custom circuit-ids:
S2#

Проверим привязку отслеживания DHCP:

S2#sh ip dhcp snooping binding 
MacAddress          IpAddress        Lease(sec)  Type           VLAN  Interface
------------------  ---------------  ----------  -------------  ----  --------------------
00:50:79:66:68:01   192.168.20.3     86389       dhcp-snooping   20    GigabitEthernet0/0
Total number of bindings: 1

S2#


