# Configure Router-on-a-Stick Inter-VLAN Routing

### Топология

![](lab1.png)

### Таблица адресации

| Устройство  | Интерфейс   | IP  -адрес          | Маска подсети  | Шлюз по умолчанию |
|-------------|-------------|---------------------|----------------|-------------------|
| R1          | e0/0.3      | 192.168.3.1         | 255.255.255.0  | -                 | 
| R1          | e0/0.4      | 192.168.4.1         | 255.255.255.0  | -                 | 
| R1          | e0/0.8      | -                   | -              | -                 | 
| S1          | VLAN 3      | 192.168.3.11        | 255.255.255.0  | 192.168.3.1       | 
| S2          | VLAN 3      | 192.168.3.12        | 255.255.255.0  | 192.168.3.1       | 
|PC-A         | NIC         | 192.168.3.3         | 255.255.255.0  | 192.168.3.1       |
|PC-B         | NIC         | 192.168.4.3         | 255.255.255.0  | 192.168.4.1       |

### Таблица VLAN

| VLAN        |    Имя       | Назначенный интерфейс                 | 
|-------------|--------------|---------------------------------------|
| 3           | Management   | S1: VLAN 3 , S2: VLAN 3, S1: F0/6     | 
| 4           | Operations   | S2: F0/18                             |  
| 7           | Parking_Lot  | S1: F0/2-4, F0/7-24, G0/1-2           |
| 8           | Native       | -                                     | 

### Часть 1. Настройка основных параметров устройств

В CPT создана лаборатория:

![](1.png)

### Шаг 1. Базовая настройка коммутаторов.

S1

```
Switch#conf t
Switch#erase startup-config 
Switch#delete vlan.dat
Switch#reload

Switch#conf t
Switch(config)#hostname S1
S1(config)#no ip domain-lookup 
S1(config)#username admin privilege 15 secret cisco
S1(config)#line con 0
S1(config-line)#login local
S1(config)#ip domain name alex.com
S1(config)#crypto key generate rsa 
The name for the keys will be: S1.alex.com
Choose the size of the key modulus in the range of 360 to 4096 for your
  General Purpose Keys. Choosing a key modulus greater than 512 may take
  a few minutes.

How many bits in the modulus [512]: 2048
% Generating 2048 bit RSA keys, keys will be non-exportable...[OK]
S1(config)#ip ssh version 2
S1(config)#ip ssh authentication-retries 3
S1(config)#line vty 0 4
S1(config-line)#login local 
S1(config-line)#transport input ssh 
S1(config-line)#exec-timeout 20 0
S1(config)#banner motd "Attention"
S1#clock set 11:10:00 18 october 2024
S1#wr

```

S2 - Аналогичная настройка

### Шаг 2. Базовая настройка маршрутизатора.

R1

```
Router#conf t
Router#erase startup-config 
Router#reload

Routerh#conf t
R1(config)#hostname S1
R1(config)#no ip domain-lookup 
R1(config)#username admin privilege 15 secret cisco
R1(config)#line con 0
R1(config-line)#login local
R1(config)#ip domain name alex.com
R1(config)#crypto key generate rsa 
The name for the keys will be: S1.alex.com
Choose the size of the key modulus in the range of 360 to 4096 for your
  General Purpose Keys. Choosing a key modulus greater than 512 may take
  a few minutes.

How many bits in the modulus [512]: 2048
% Generating 2048 bit RSA keys, keys will be non-exportable...[OK]
R1(config)#ip ssh version 2
R1(config)#ip ssh authentication-retries 3
R1(config)#line vty 0 4
R1(config-line)#login local 
R1(config-line)#transport input ssh 
R1(config-line)#exec-timeout 20 0
R1(config)#banner motd "Attention"
R1#clock set 11:23:00 18 october 2024
R1#wr

```

### Шаг 2. Настройка ПК.

Назначим ip адреса PC-A и PC-B:

PC-A

```
C:\>ipconfig

FastEthernet0 Connection:(default port)

   Connection-specific DNS Suffix..: 
   Link-local IPv6 Address.........: FE80::206:2AFF:FEC6:6A18
   IPv6 Address....................: ::
   IPv4 Address....................: 192.168.20.3
   Subnet Mask.....................: 255.255.255.0
   Default Gateway.................: ::
                                     192.168.20.1
C:\>
```

PC-B

```
C:\>ipconfig

FastEthernet0 Connection:(default port)

   Connection-specific DNS Suffix..: 
   Link-local IPv6 Address.........: FE80::2E0:F9FF:FEAD:3851
   IPv6 Address....................: ::
   IPv4 Address....................: 192.168.30.3
   Subnet Mask.....................: 255.255.255.0
   Default Gateway.................: ::
                                     192.168.30.1
C:\>

```

### Часть 2. Создание сетей VLAN и назначение портов коммутатора.

Создадим VLAN, как указано в таблице выше, на обоих коммутаторах. Затем назначим VLAN соответствующему интерфейсу и проверим настройки конфигурации.

### Шаг 1. Создаим сети VLAN на коммутаторах.

Создаим VLAN на каждом коммутаторе из таблицы выше.

```
S1#conf t	
S1(config)#vlan 10 
S1(config-vlan)#name mng
S1(config-vlan)#vlan 20
S1(config-vlan)#name sales
S1(config-vlan)#vlan 30
S1(config-vlan)#name operations
S1(config-vlan)#vlan 999
S1(config-vlan)#name parkin_lot
S1(config)#vlan 1000
S1(config-vlan)#name native
```

Настроим интерфейс управления и шлюз по умолчанию на каждом коммутаторе, используя информацию об IP-адресе в таблице адресации.

```
S1(config)#int vlan 10
S1(config-if)#
%LINK-5-CHANGED: Interface Vlan10, changed state to up
S1(config-if)#ip address 192.168.10.11 255.255.255.0
S1(config)#ip default-gateway 192.168.10.1
```

Назначим все неиспользуемые порты коммутатора в VLAN Parking_Lot, настроим их для статического режима доступа и административно деактивируем их.

```
S1(config)#int range f 0/2-4 , f 0/7-24 , g 0/1-2
S1(config-if-range)#switchport mode access 
S1(config-if-range)#switchport access vlan 999
S1(config-if-range)#switchport nonegotiate 
S1(config-if-range)#shutdown 
```

Конфиг для S2:

```
S2#conf t
S2(config)#vlan 10
S2(config-vlan)#name mng
S2(config-vlan)#vlan 20
S2(config-vlan)#name sales
S2(config-vlan)#vlan 30
S2(config-vlan)#name operations
S2(config-vlan)#vlan 999
S2(config-vlan)#name parking_lot
S1(config)#vlan 1000
S1(config-vlan)#name native
S2(config)#int vlan 10
S2(config-if)#
%LINK-5-CHANGED: Interface Vlan10, changed state to up
S2(config-if)#ip address 192.168.10.12 255.255.255.0
S2(config)#ip default-gateway 192.168.10.1
S2(config)#int range f 0/2-17 , f0/19-24 , g 0/1-2
S2(config-if-range)#switchport mode access 
S2(config-if-range)#switchport access vlan 999
S2(config-if-range)#switchport nonegotiate 
S2(config-if-range)#shutdown 
```


### Шаг 2. Назначим сети VLAN соответствующим интерфейсам коммутатора.

Назначим используемые порты соответствующей VLAN (указанной в таблице VLAN выше) и настроим их для режима статического доступа.

S1

```
S1(config)#int f 0/6
S1(config-if)#switchport mode access 
S1(config-if)#switchport access vlan 20
S1(config-if)#no shutdown 
```

S2

```
S2(config)#int f 0/18
S2(config-if)#switchport mode access 
S2(config-if)#switchport access vlan 30
S2(config-if)#no shutdown 
```

Убедимся, что VLAN назначены на правильные интерфейсы.

S1:

```
S1#sh vlan brief 

VLAN Name                             Status    Ports
---- -------------------------------- --------- -------------------------------
1    default                          active    Fa0/5
10   mng                              active    
20   sales                            active    Fa0/6
30   operations                       active    
999  parkin_lot                       active    Fa0/2, Fa0/3, Fa0/4, Fa0/7
                                                Fa0/8, Fa0/9, Fa0/10, Fa0/11
                                                Fa0/12, Fa0/13, Fa0/14, Fa0/15
                                                Fa0/16, Fa0/17, Fa0/18, Fa0/19
                                                Fa0/20, Fa0/21, Fa0/22, Fa0/23
                                                Fa0/24, Gig0/1, Gig0/2
1000 native                           active    
1002 fddi-default                     active    
1003 token-ring-default               active    
1004 fddinet-default                  active    
1005 trnet-default                    active    
S1#
```

S2:

```
S2#sh vlan brief 

VLAN Name                             Status    Ports
---- -------------------------------- --------- -------------------------------
1    default                          active    
10   mng                              active    
20   sales                            active    
30   operations                       active    Fa0/18
999  parking_lot                      active    Fa0/2, Fa0/3, Fa0/4, Fa0/5
                                                Fa0/6, Fa0/7, Fa0/8, Fa0/9
                                                Fa0/10, Fa0/11, Fa0/12, Fa0/13
                                                Fa0/14, Fa0/15, Fa0/16, Fa0/17
                                                Fa0/19, Fa0/20, Fa0/21, Fa0/22
                                                Fa0/23, Fa0/24, Gig0/1, Gig0/2
1000 native                           active    
1002 fddi-default                     active    
1003 token-ring-default               active    
1004 fddinet-default                  active    
1005 trnet-default                    active    
S2#
```

Все настроено верно.

### Часть 3. Конфигурация магистрального канала стандарта 802.1Q между коммутаторами.

### Шаг 1. Вручную настроим магистральный интерфейс F0/1 на коммутаторах S1 и S2.

Настроим статический транкинг на интерфейсе F0/1 для обоих коммутаторов.

```
S1(config)#int f 0/1
S1(config-if)#switchport mode trunk 
```

Установим native VLAN 1000 на обоих коммутаторах.

```
S1(config-if)#switchport trunk native vlan 1000
```

Укажим, что VLAN 10, 20, 30 и 1000 могут проходить по транку.

```
S1(config-if)#switchport trunk allowed vlan 10,20,30,1000
```

Аналогичная настройка будет на S2. 


Далее проверим транки, native VLAN и разрешенные VLAN через транк.

S1:

```
S1#sh interfaces trunk 
Port        Mode         Encapsulation  Status        Native vlan
Fa0/1       on           802.1q         trunking      1000

Port        Vlans allowed on trunk
Fa0/1       10,20,30,1000

Port        Vlans allowed and active in management domain
Fa0/1       10,20,30,1000

Port        Vlans in spanning tree forwarding state and not pruned
Fa0/1       10,20,30,1000

S1#
```

S2:

```
S2#sh int trunk 
Port        Mode         Encapsulation  Status        Native vlan
Fa0/1       on           802.1q         trunking      1000

Port        Vlans allowed on trunk
Fa0/1       10,20,30,1000

Port        Vlans allowed and active in management domain
Fa0/1       10,20,30,1000

Port        Vlans in spanning tree forwarding state and not pruned
Fa0/1       10,20,30,1000

S2#
```

### Шаг 2. Вручную настроим магистральный интерфейс F0/5 на коммутаторе S1.

Настроим транк до маршрутизатора:

```
S1(config)#int f 0/5
S1(config-if)#switchport mode trunk                                       ^
S1(config-if)#switchport trunk allowed vlan 10,20,30,1000
S1(config-if)#switchport trunk native vlan 1000
S1(config-if)#do copy run start
```

Проверка транкинга.

```
S1#sh int trunk 
Port        Mode         Encapsulation  Status        Native vlan
Fa0/1       on           802.1q         trunking      1000

Port        Vlans allowed on trunk
Fa0/1       10,20,30,1000

Port        Vlans allowed and active in management domain
Fa0/1       10,20,30,1000

Port        Vlans in spanning tree forwarding state and not pruned
Fa0/1       10,20,30,1000

S1#
```

Из вывода данной команды видно, что порт f 0/5 отсутствует, т.к он находится в состоянии down (FastEthernet0/5 is down, line protocol is down (disabled)). Отсутствует настройка на стороне маршрутизатора.

```
S1#sh int f 0/5 status 
Port      Name               Status       Vlan       Duplex  Speed Type
Fa0/5                        notconnect   trunk      auto    auto  10/100BaseTX
```

проверка настроек порта:

```
S1#sh int f 0/5 switchport 
Name: Fa0/5
Switchport: Enabled
Administrative Mode: trunk
Operational Mode: down
Administrative Trunking Encapsulation: dot1q
Operational Trunking Encapsulation: dot1q
Negotiation of Trunking: On
Access Mode VLAN: 1 (default)
Trunking Native Mode VLAN: 1000 (native)
Voice VLAN: none
Administrative private-vlan host-association: none
Administrative private-vlan mapping: none
Administrative private-vlan trunk native VLAN: none
Administrative private-vlan trunk encapsulation: dot1q
Administrative private-vlan trunk normal VLANs: none
Administrative private-vlan trunk private VLANs: none
Operational private-vlan: none
Trunking VLANs Enabled: 10,20,30,1000
Pruning VLANs Enabled: 2-1001
Capture Mode Disabled
Capture VLANs Allowed: ALL
Protected: false
Unknown unicast blocked: disabled
Unknown multicast blocked: disabled
Appliance trust: none
```

При отсутствии транка в сторону маршрутизатора, ip связанность будет только в рамках одной vlan.

### Часть 4. Настройка маршрутизации между сетями VLAN.

Активируем интерфейс G0/0/1 на маршрутизаторе

Настроим подинтерфейсы для каждой VLAN, как указано в таблице IP-адресации. Все подинтерфейсы используют инкапсуляцию 802.1Q. Убедимся, что подинтерфейсу для native VLAN не назначен IP-адрес. Включим описание для каждого подинтерфейса.

```
R1(config)#int g 0/0/1
R1(config-if)#no shutdown 
R1(config-if)#int g 0/0/1.10
R1(config-subif)#description mng
R1(config-subif)#encapsulation dot1Q 10
R1(config-subif)#ip address 192.168.10.1 255.255.255.0
R1(config-subif)#int g 0/0/1.20
R1(config-subif)#description sales
R1(config-subif)#encapsulation dot1Q 20
R1(config-subif)#ip address 192.168.20.1 255.255.255.0
R1(config-subif)#int g 0/0/1.30
R1(config-subif)#description operations
R1(config-subif)#encapsulation dot1Q 30
R1(config-subif)#ip address 192.168.30.1 255.255.255.0
R1(config-subif)#int g 0/0/1.1000
R1(config-subif)#encapsulation dot1Q 1000 native 
```

Проверим настройки саб интерфейсов:

```
R1#sh ip int brief 
Interface              IP-Address      OK? Method Status                Protocol 
GigabitEthernet0/0/0   unassigned      YES NVRAM  administratively down down 
GigabitEthernet0/0/1   unassigned      YES NVRAM  up                    up 
GigabitEthernet0/0/1.10 192.168.10.1    YES manual up                    up 
GigabitEthernet0/0/1.20 192.168.20.1    YES manual up                    up 
GigabitEthernet0/0/1.30 192.168.30.1    YES manual up                    up 
GigabitEthernet0/0/1.1000 unassigned      YES unset  up                    up 
GigabitEthernet0/0/2   unassigned      YES NVRAM  administratively down down 
Vlan1                  unassigned      YES NVRAM  administratively down down
R1#
```

### Часть 5. Проверим, работает ли маршрутизация между VLAN

Отправим эхо-запрос с PC-A на шлюз по умолчанию.

```
C:\>ping 192.168.20.1

Pinging 192.168.20.1 with 32 bytes of data:

Reply from 192.168.20.1: bytes=32 time<1ms TTL=255
Reply from 192.168.20.1: bytes=32 time<1ms TTL=255
Reply from 192.168.20.1: bytes=32 time<1ms TTL=255
Reply from 192.168.20.1: bytes=32 time<1ms TTL=255

Ping statistics for 192.168.20.1:
    Packets: Sent = 4, Received = 4, Lost = 0 (0% loss),
Approximate round trip times in milli-seconds:
    Minimum = 0ms, Maximum = 0ms, Average = 0ms

C:\>
```

Отправим эхо-запрос с PC-A на PC-B.

```
C:\>ping 192.168.30.1

Pinging 192.168.30.1 with 32 bytes of data:

Reply from 192.168.30.1: bytes=32 time=12ms TTL=255
Reply from 192.168.30.1: bytes=32 time<1ms TTL=255
Reply from 192.168.30.1: bytes=32 time<1ms TTL=255
Reply from 192.168.30.1: bytes=32 time<1ms TTL=255

Ping statistics for 192.168.30.1:
    Packets: Sent = 4, Received = 4, Lost = 0 (0% loss),
Approximate round trip times in milli-seconds:
    Minimum = 0ms, Maximum = 12ms, Average = 3ms

C:\>
```

Отправим команду ping с компьютера PC-A на коммутатор S2.

```
C:\>ping 192.168.10.12

Pinging 192.168.10.12 with 32 bytes of data:

Reply from 192.168.10.12: bytes=32 time<1ms TTL=254
Reply from 192.168.10.12: bytes=32 time<1ms TTL=254
Reply from 192.168.10.12: bytes=32 time<1ms TTL=254
Reply from 192.168.10.12: bytes=32 time<1ms TTL=254

Ping statistics for 192.168.10.12:
    Packets: Sent = 4, Received = 4, Lost = 0 (0% loss),
Approximate round trip times in milli-seconds:
    Minimum = 0ms, Maximum = 0ms, Average = 0ms

C:\>
```

На PC-B выполним команду tracert на адрес PC-A.

```
C:\>tracert 192.168.20.3

Tracing route to 192.168.20.3 over a maximum of 30 hops: 

  1   0 ms      0 ms      0 ms      192.168.30.1
  2   0 ms      0 ms      0 ms      192.168.20.3

Trace complete.
