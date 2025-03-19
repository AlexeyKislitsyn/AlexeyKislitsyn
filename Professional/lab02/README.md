
## Развертывание коммутируемой сети с резервными каналами 

#### Топология

![](lab02.png)

### Таблица адресации

| Устройство  | Интерфейс   | IP  -адрес   | Маска подсети  | 
|-------------|-------------|--------------|----------------|
| S1          | VLAN 1      | 192.168.1.1  | 255.255.255.0  | 
| S2          | VLAN 1      | 192.168.1.2  | 255.255.255.0  |  
| S3          | VLAN 1      | 192.168.1.3  | 255.255.255.0  |

#### Часть 1. Настройка основных параметров устройств

#### Шаг 1. Базовая настройка коммутаторов.

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
S1(config-line)#logging synchronous
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
S1(config)#int vlan 1
S1(config-if)#ip address 192.168.1.1 255.255.255.0
S1(config-if)#no shutdown 
S1#copy run start

```

S2, S3 - Аналогичная настройка, исключением является настройка ip адреса vlan 1 согласно таблицы адресации.

### Шаг 2: Создадим vlan на всех 3-х коммутаторах:

```
VLAN Name                             Status    Ports
---- -------------------------------- --------- -------------------------------
1    default                          active    
20   USER_20                          active    
30   USER_30                          active    
40   USER_40                          active    
50   USER_50                          active    
100  MANAGEMENT                       active    
999  NATIVE                           active    
```

### Шаг 3: Линки между коммутаторами переведем в транк:

S2:

```
interface Ethernet0/0
 switchport trunk encapsulation dot1q
 switchport trunk native vlan 999
 switchport mode trunk
!
interface Ethernet0/1
 switchport trunk encapsulation dot1q
 switchport trunk native vlan 999
 switchport mode trunk
!
interface Ethernet0/2
 switchport trunk encapsulation dot1q
 switchport trunk native vlan 999
 switchport mode trunk
!
interface Ethernet0/3
 switchport trunk encapsulation dot1q
 switchport trunk native vlan 999
 switchport mode trunk
```



### Часть 2: Настроим регион: имя, ревизия, маппинг.

S2(config)#spanning-tree mode mst 
S2(config)#spanning-tree mst configuration 
S2(config-mst)#name REGION_1
S2(config-mst)#revision 1
S2(config-mst)#instance 1 vlan 20,30,100,999 
S2(config-mst)#instance 2 vlan 40,50

Аналогичные настройки выполним на S1 и S3


Введем команду show spanning-tree mst на всех трех коммутаторах. Приоритет идентификатора моста рассчитывается путем сложения значений приоритета и расширенного идентификатора системы. Расширенным идентификатором системы всегда является номер сети VLAN. В примере ниже все три коммутатора имеют равные значения приоритета идентификатора моста (приоритет по умолчанию = 32768,+ номер VLAN); следовательно, коммутатор с самым низким значением MAC-адреса становится корневым мостом, в данном случае S1.

```
S1#sh spanning-tree mst 

##### MST0    vlans mapped:   1-19,21-29,31-39,41-49,51-99,101-998,1000-4094
Bridge        address aabb.cc00.1000  priority      32768 (32768 sysid 0)
Root          this switch for the CIST
Operational   hello time 2 , forward delay 15, max age 20, txholdcount 6 
Configured    hello time 2 , forward delay 15, max age 20, max hops    20

Interface        Role Sts Cost      Prio.Nbr Type
---------------- ---- --- --------- -------- --------------------------------
Et0/0            Desg FWD 2000000   128.1    P2p 
Et0/1            Desg FWD 2000000   128.2    P2p 
Et0/2            Desg FWD 2000000   128.3    P2p 
Et0/3            Desg FWD 2000000   128.4    P2p 

##### MST1    vlans mapped:   20,30,100,999
Bridge        address aabb.cc00.1000  priority      32769 (32768 sysid 1)
Root          this switch for MST1

Interface        Role Sts Cost      Prio.Nbr Type
---------------- ---- --- --------- -------- --------------------------------
Et0/0            Desg FWD 2000000   128.1    P2p 
Et0/1            Desg FWD 2000000   128.2    P2p 
Et0/2            Desg FWD 2000000   128.3    P2p 
Et0/3            Desg FWD 2000000   128.4    P2p 

##### MST2    vlans mapped:   40,50
Bridge        address aabb.cc00.1000  priority      32770 (32768 sysid 2)
Root          this switch for MST2

Interface        Role Sts Cost      Prio.Nbr Type
---------------- ---- --- --------- -------- --------------------------------
Et0/0            Desg FWD 2000000   128.1    P2p 
Et0/1            Desg FWD 2000000   128.2    P2p 
Et0/2            Desg FWD 2000000   128.3    P2p 
Et0/3            Desg FWD 2000000   128.4    P2p 

S1#

```


### Шаг 4: Настройка корневого моста.

Очень желательно указывать рутовый свич руками. Без такого указания взял и рутом выбрался коммутатор доступа, чего быть не должно. Пока один рут, а он один и тот же для всех VLANs, топология всех VLANs в нашем конкретном случае получилась одинаковая. И то что мы распихали их по двум инстансам результата не даст. Рутовые, назначенные и альтернативные порты одни и те же. Поэтому и нужно определять VLANs с одинаковой топологией в один инстанс.

Корневой мост настраивается как в классическом STP, только вместо VLAN указывается инстанс.

Пусть для vlan 20,30,100,999 (MST1) корневым мостом будет S1, а для vlan 40,50 (MST2) корневым мостом будет S2.

S1: 

```
S1(config)#spanning-tree mst 1 root primary 
S1(config)#spanning-tree mst 2 root second  
```

S2: 

```
S2(config)#spanning-tree mst 1 root sec
S2(config)#spanning-tree mst 2 root prim

```

Таким образом у нас пполучается следующая картина:

```
S1#sh spanning-tree mst 

##### MST0    vlans mapped:   1-19,21-29,31-39,41-49,51-99,101-998,1000-4094
Bridge        address aabb.cc00.1000  priority      32768 (32768 sysid 0)
Root          this switch for the CIST
Operational   hello time 2 , forward delay 15, max age 20, txholdcount 6 
Configured    hello time 2 , forward delay 15, max age 20, max hops    20

Interface        Role Sts Cost      Prio.Nbr Type
---------------- ---- --- --------- -------- --------------------------------
Et0/0            Desg FWD 2000000   128.1    P2p 
Et0/1            Desg FWD 2000000   128.2    P2p 
Et0/2            Desg FWD 2000000   128.3    P2p 
Et0/3            Desg FWD 2000000   128.4    P2p 

##### MST1    vlans mapped:   20,30,100,999
Bridge        address aabb.cc00.1000  priority      24577 (24576 sysid 1)
Root          this switch for MST1

Interface        Role Sts Cost      Prio.Nbr Type
---------------- ---- --- --------- -------- --------------------------------
Et0/0            Desg FWD 2000000   128.1    P2p 
Et0/1            Desg FWD 2000000   128.2    P2p 
Et0/2            Desg FWD 2000000   128.3    P2p 
Et0/3            Desg FWD 2000000   128.4    P2p 

##### MST2    vlans mapped:   40,50
Bridge        address aabb.cc00.1000  priority      28674 (28672 sysid 2)
Root          address aabb.cc00.2000  priority      24578 (24576 sysid 2)
              port    Et0/0           cost          2000000   rem hops 19

Interface        Role Sts Cost      Prio.Nbr Type
---------------- ---- --- --------- -------- --------------------------------
Et0/0            Root FWD 2000000   128.1    P2p 
Et0/1            Altn BLK 2000000   128.2    P2p 
Et0/2            Desg FWD 2000000   128.3    P2p 
Et0/3            Desg FWD 2000000   128.4    P2p 

S1#
```

### Часть 3:	Наблюдение за процессом выбора протоколом STP порта, исходя из стоимости портов

Порт F0/4 коммутатора S3 альтернативный и заблокирован.

### Шаг 1:	Изменим стоимость порта.

Помимо заблокированного порта, единственным активным портом на этом коммутаторе является порт, выделенный в качестве порта корневого моста. Уменьшим стоимость этого порта корневого моста до 18

```
S3(config)#int f 0/2
S3(config-if)#spanning-tree vlan 1 cost 18
```

### Шаг 2:	Посмотрим изменения протокола spanning-tree.

S1:

```
S1#sh spanning-tree 
VLAN0001
  Spanning tree enabled protocol ieee
  Root ID    Priority    32769
             Address     0010.11B7.C60E
             Cost        19
             Port        2(FastEthernet0/2)
             Hello Time  2 sec  Max Age 20 sec  Forward Delay 15 sec

  Bridge ID  Priority    32769  (priority 32768 sys-id-ext 1)
             Address     0060.2F46.1DC9
             Hello Time  2 sec  Max Age 20 sec  Forward Delay 15 sec
             Aging Time  20

Interface        Role Sts Cost      Prio.Nbr Type
---------------- ---- --- --------- -------- --------------------------------
Fa0/4            Altn BLK 19        128.4    P2p
Fa0/2            Root FWD 19        128.2    P2p

S1#
```

S3:

```
S3#sh spanning-tree 
VLAN0001
  Spanning tree enabled protocol ieee
  Root ID    Priority    32769
             Address     0010.11B7.C60E
             Cost        18
             Port        2(FastEthernet0/2)
             Hello Time  2 sec  Max Age 20 sec  Forward Delay 15 sec

  Bridge ID  Priority    32769  (priority 32768 sys-id-ext 1)
             Address     00D0.BC93.D827
             Hello Time  2 sec  Max Age 20 sec  Forward Delay 15 sec
             Aging Time  20

Interface        Role Sts Cost      Prio.Nbr Type
---------------- ---- --- --------- -------- --------------------------------
Fa0/2            Root FWD 18        128.2    P2p
Fa0/4            Desg FWD 19        128.4    P2p

S3#
```

Т.к мы изменили стоимость порта root на S3 в меньшую сторону , то Fa0/4 на S1 стал Altn-BLK, а Fa0/4 S3 изменился на Desg. 

### Шаг 3:	Удалим изменения стоимости порт.

```
S3(config)#int f 0/2
S3(config-if)#no spanning-tree vlan 1 cost 18
S3(config-if)#
```

После выполнения данной команды, протокол STP сбросил порт на коммутаторе некорневого моста, вернув исходные настройки порта.

### Часть 4:	Наблюдение за процессом выбора протоколом STP порта, исходя из приоритета портов

Включим порты F0/1 и F0/3 на всех коммутаторах.

```
S3(config)#int range f0/1, f0/3
S3(config-if-range)#no shutdown

S2(config)#int ran f0/1, f0/3
S2(config-if-range)#no shutdown 

S1(config)#int range f0/1, f0/3
S1(config-if-range)#no sh
```

S1:

```
S1# sh spanning-tree 
VLAN0001
  Spanning tree enabled protocol ieee
  Root ID    Priority    32769
             Address     0010.11B7.C60E
             Cost        19
             Port        1(FastEthernet0/1)
             Hello Time  2 sec  Max Age 20 sec  Forward Delay 15 sec

  Bridge ID  Priority    32769  (priority 32768 sys-id-ext 1)
             Address     0060.2F46.1DC9
             Hello Time  2 sec  Max Age 20 sec  Forward Delay 15 sec
             Aging Time  20

Interface        Role Sts Cost      Prio.Nbr Type
---------------- ---- --- --------- -------- --------------------------------
Fa0/4            Desg FWD 19        128.4    P2p
Fa0/3            Desg FWD 19        128.3    P2p
Fa0/1            Root FWD 19        128.1    P2p
Fa0/2            Altn BLK 19        128.2    P2p

S1#
```

S2:

```
S2#sh spanning-tree 
VLAN0001
  Spanning tree enabled protocol ieee
  Root ID    Priority    32769
             Address     0010.11B7.C60E
             This bridge is the root
             Hello Time  2 sec  Max Age 20 sec  Forward Delay 15 sec

  Bridge ID  Priority    32769  (priority 32768 sys-id-ext 1)
             Address     0010.11B7.C60E
             Hello Time  2 sec  Max Age 20 sec  Forward Delay 15 sec
             Aging Time  20

Interface        Role Sts Cost      Prio.Nbr Type
---------------- ---- --- --------- -------- --------------------------------
Fa0/3            Desg FWD 19        128.3    P2p
Fa0/1            Desg FWD 19        128.1    P2p
Fa0/2            Desg FWD 19        128.2    P2p
Fa0/4            Desg FWD 19        128.4    P2p

S2#
```

S3:
```
S3#sh spanning-tree 
VLAN0001
  Spanning tree enabled protocol ieee
  Root ID    Priority    32769
             Address     0010.11B7.C60E
             Cost        19
             Port        1(FastEthernet0/1)
             Hello Time  2 sec  Max Age 20 sec  Forward Delay 15 sec

  Bridge ID  Priority    32769  (priority 32768 sys-id-ext 1)
             Address     00D0.BC93.D827
             Hello Time  2 sec  Max Age 20 sec  Forward Delay 15 sec
             Aging Time  20

Interface        Role Sts Cost      Prio.Nbr Type
---------------- ---- --- --------- -------- --------------------------------
Fa0/1            Root FWD 19        128.1    P2p
Fa0/2            Altn BLK 19        128.2    P2p
Fa0/3            Altn BLK 19        128.3    P2p
Fa0/4            Altn BLK 19        128.4    P2p

S3#
```

Порты f0/1 на S1 и S3 выбраны как root. Если стоимости портов равны, процесс сравнивает BID. Если BID равны, для определения корневого моста используются приоритеты портов. Значение приоритета по умолчанию — 128. STP объединяет приоритет порта с номером порта, чтобы разорвать связи. Наиболее низкие значения являются предпочтительными. Таким образом Prio.Nbr - 128.1 - предпочтительней.
