## OSPF. Фильтрация

#### Цель:

Настроить OSPF офисе Москва Разделить сеть на зоны Настроить фильтрацию между зонами

1. Маршрутизаторы R14-R15 находятся в зоне 0 - backbone
2. Маршрутизаторы R12-R13 находятся в зоне 10. Дополнительно к маршрутам должны получать маршрут по-умолчанию
3. Маршрутизатор R19 находится в зоне 101 и получает только маршрут по умолчанию
4. Маршрутизатор R20 находится в зоне 102 и получает все маршруты, кроме маршрутов до сетей зоны 1015
5. План работы и изменения зафиксированы в документации

![](Moskow1.png)

#### Выполнение.

*В Московском офисе у нас 3-х уровневая модель, поэтому:

-  area 0 – будет Core layer
-  area 10 – будет реализован Distribution layer

Доолнительно, для построения отказоустойчивой сети, добавим линки между R14-R15 и R12-R13 (L3 коммутаторы). Запустим процесс OSPF на всех маршрутизаторах.

* Типичная настройка представлена для R15:

```
!
interface Loopback0
 ip address 10.10.11.2 255.255.255.255
 ip ospf 1 area 0
!         
interface Ethernet0/0
 description TO_R13
 ip address 10.0.0.14 255.255.255.252
 ip ospf network point-to-point
 ip ospf dead-interval 12
 ip ospf hello-interval 3
 ip ospf 1 area 0
!
interface Ethernet0/1
 description TO_R12
 ip address 10.0.0.6 255.255.255.252
 ip ospf network point-to-point
 ip ospf dead-interval 12
 ip ospf hello-interval 3
 ip ospf 1 area 0
!
interface Ethernet0/2
 description TO_R21_LAMAS
 ip address 30.1.1.2 255.255.255.248
!
interface Ethernet0/3
 description TO_R20
 ip address 10.0.3.1 255.255.255.252
 ip ospf network point-to-point
 ip ospf dead-interval 12
 ip ospf hello-interval 3
 ip ospf 1 area 102
!
interface Ethernet1/0
 description TO_R14
 ip address 10.0.0.18 255.255.255.252
 ip ospf network point-to-point
 ip ospf dead-interval 12
 ip ospf hello-interval 3
 ip ospf 1 area 0
!
interface Ethernet1/1
 no ip address
 shutdown
!
interface Ethernet1/2
 no ip address
 shutdown
!
interface Ethernet1/3
 no ip address
 shutdown
!
router ospf 1
 router-id 15.15.15.15
 passive-interface default
 no passive-interface Ethernet0/0
 no passive-interface Ethernet0/1
 no passive-interface Ethernet0/3
 no passive-interface Ethernet1/0
!
```
* Настройка для L3 коммутатора на примере SW5:

```
!
interface Loopback0
 ip address 10.10.10.2 255.255.255.255
 ip ospf 1 area 10
!
interface Port-channel1
 description TO_SW4
 no switchport
 ip address 10.0.1.2 255.255.255.252
 ip ospf network point-to-point
 ip ospf dead-interval 12
 ip ospf hello-interval 3
 ip ospf 1 area 10
!
interface Ethernet0/0
 description TRUNK_TO_SW2
 switchport trunk allowed vlan 10,20,100,1000
 switchport trunk encapsulation dot1q
 switchport trunk native vlan 1000
 switchport mode trunk
!
interface Ethernet0/1
 description TRUNK_TO_SW3
 switchport trunk allowed vlan 10,20,100,1000
 switchport trunk encapsulation dot1q
 switchport trunk native vlan 1000
 switchport mode trunk
!
interface Ethernet0/2
 no switchport
 no ip address
 channel-group 1 mode active
!
interface Ethernet0/3
 no switchport
 no ip address
 channel-group 1 mode active
!
interface Ethernet1/0
 description TO_R13
 no switchport
 ip address 10.0.1.17 255.255.255.252
 ip ospf network point-to-point
 ip ospf dead-interval 12
 ip ospf hello-interval 3
 ip ospf 1 area 10
!
interface Ethernet1/1
 description TO_R12
 no switchport
 ip address 10.0.1.13 255.255.255.252
 ip ospf network point-to-point
 ip ospf dead-interval 12
 ip ospf hello-interval 3
 ip ospf 1 area 10
!
interface Ethernet1/2
!
interface Ethernet1/3
!
interface Vlan10
 description VL10_USER
 ip address 192.168.1.254 255.255.255.0
 standby version 2
 standby 10 ip 192.168.1.1
 ip ospf 1 area 10
!
interface Vlan20
 description VL20_USER
 ip address 192.168.2.254 255.255.255.0
 standby version 2
 standby 20 ip 192.168.2.1
 standby 20 priority 150
 standby 20 preempt
 ip ospf 1 area 10
!
interface Vlan100
 description MNG
 ip address 192.168.100.126 255.255.255.128
 standby version 2
 standby 100 ip 192.168.100.1
 ip ospf 1 area 10
!
router ospf 1
 router-id 5.5.5.5
 passive-interface default
 no passive-interface Ethernet1/0
 no passive-interface Ethernet1/1
 no passive-interface Port-channel1
!
```
