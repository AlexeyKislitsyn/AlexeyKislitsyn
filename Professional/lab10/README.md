
## iBGP.

#### Цель:

Настроить iBGP в офисе Москва

Настроить iBGP в сети провайдера Триада

Организовать полную IP связанность всех сетей

#### План раоты

1. Настроить iBGP в офисом Москва между маршрутизаторами R14 и R15.
2. Настроить iBGP в провайдере Триада.
3. Настроить офиса Москва так, чтобы приоритетным провайдером стал Ламас.
4. Настроить офиса С.-Петербург так, чтобы трафик до любого офиса распределялся по двум линкам одновременно.
5. Все сети в лабораторной работе должны иметь IP связность.


##  Схема стенда 

![](ibgp.png)

#### Выполнение.

#### Настроика iBGP в офисе Москва между маршрутизаторами R14 и R15.

Номер AS AS1001 Москва и блок адресов: 140.100.0.0/23. Разобьем наш блок на 2 подсети. R14 эмулирует 140.100.0.0/24, R15 - 140.100.1.0/24. Настроены на loopback 100 интерфейсах R14 и R15.

Согласно схемы, на R14 и R15 настроены loopback 0 интерфейсы, которые добавлены в ospf процесс. Между R14 и R15 iBGP сессию будет поднимать от loopback 0 интерфейсов.

  
```
### R14:

interface Loopback0
 ip address 10.10.11.1 255.255.255.255
 ip ospf 1 area 0
end

R14#sh run | s r b 
router bgp 1001
 bgp router-id 14.14.14.14
 bgp log-neighbor-changes
 network 140.100.0.0 mask 255.255.254.0
 network 140.100.0.0 mask 255.255.255.0
 network 140.100.1.0 mask 255.255.255.0
 neighbor 10.10.11.2 remote-as 1001
 neighbor 10.10.11.2 update-source Loopback0
 neighbor 10.10.11.2 next-hop-self
 neighbor 101.0.0.1 remote-as 101
R14#


### R15:
interface Loopback0
 ip address 10.10.11.2 255.255.255.255
 ip ospf 1 area 0
end

R15#sh run | s r b
router bgp 1001
 bgp router-id 15.15.15.15
 bgp log-neighbor-changes
 network 140.100.0.0 mask 255.255.254.0
 network 140.100.0.0 mask 255.255.255.0
 network 140.100.1.0 mask 255.255.255.0
 neighbor 10.10.11.1 remote-as 1001
 neighbor 10.10.11.1 update-source Loopback0
 neighbor 10.10.11.1 next-hop-self
 neighbor 30.1.0.1 remote-as 301
R15#
```

* На R14 проверим установленные отношения соседства и таблицу маршрутизации bgp:

```
R14#sh ip bgp summary 
Neighbor        V           AS MsgRcvd MsgSent   TblVer  InQ OutQ Up/Down  State/PfxRcd
10.10.11.2      4         1001      43      56        9    0    0 00:33:11        5
101.0.0.1       4          101      47      42        9    0    0 00:33:13        4
R14#

R14#sh ip bgp 
BGP table version is 9, local router ID is 14.14.14.14
     Network          Next Hop            Metric LocPrf Weight Path
 * i  20.0.0.0/24      10.10.11.2               0    100      0 301 520 2042 i
 *>                    101.0.0.1                              0 101 520 2042 i
 *>i  30.1.0.0/22      10.10.11.2               0    100      0 301 i
 *                     101.0.0.1                              0 101 301 i
 * i  52.1.0.0/20      10.10.11.2               0    100      0 301 520 i
 *>                    101.0.0.1                              0 101 520 i
 *>   101.0.0.0/21     101.0.0.1                0             0 101 i
 *>   140.100.0.0/24   0.0.0.0                  0         32768 i
 *>i  140.100.0.0/23   10.10.11.2               0    100      0 i
 *>i  140.100.1.0/24   10.10.11.2               0    100      0 i
R14#
```





#### Настройка iBGP в Триаде.

* 1. В данной схеме необходима настройка Full Mesh. Для каждого нового члена Full Mesh необходимо дублировать настройки. Чтобы упростить подключение новых пиров – объединим их в peer-group.
* 2. Хорошо если в сети 4 маршрутизатора, а если их десятки?. Строить Full Mesh будет однозначно трудоемко. Логично будет использовать Route Reflector. В качестве RR server будет R23. R24,R25,R26 - клиенты.
* 3. Т.к у нас в сети один RR - R23 - он является единой точкой отказа. В этом сслучае настроим второй RR.

Таким образом настройка iBGP в сети Триада имеет вид:


```
R23#sh run | s r b
router bgp 520
 bgp router-id 23.23.23.23
 bgp cluster-id 1.1.1.1
 bgp log-neighbor-changes
 network 52.1.0.0 mask 255.255.240.0
 neighbor RR1-TRIADA peer-group
 neighbor RR1-TRIADA remote-as 520
 neighbor RR1-TRIADA update-source Loopback0
 neighbor RR1-TRIADA route-reflector-client
 neighbor 10.10.10.24 peer-group RR1-TRIADA
 neighbor 10.10.10.25 peer-group RR1-TRIADA
 neighbor 10.10.10.26 peer-group RR1-TRIADA
 neighbor 52.1.0.13 remote-as 101
R23#

R23#sh ip bgp summary 
BGP router identifier 23.23.23.23, local AS number 520
Neighbor        V           AS MsgRcvd MsgSent   TblVer  InQ OutQ Up/Down  State/PfxRcd
10.10.10.24     4          520      32      37       26    0    0 00:22:57        6
10.10.10.25     4          520      77      94       26    0    0 01:08:42        0
10.10.10.26     4          520      76      93       26    0    0 01:04:32        1
52.1.0.13       4          101      91      88       26    0    0 01:08:48        5
R23#


R24#sh run | s r b
router bgp 520
 bgp router-id 24.24.24.24
 bgp cluster-id 1.1.1.1
 bgp log-neighbor-changes
 network 52.1.0.0 mask 255.255.240.0
 neighbor RR2-TRIADA peer-group
 neighbor RR2-TRIADA remote-as 520
 neighbor RR2-TRIADA update-source Loopback0
 neighbor RR2-TRIADA route-reflector-client
 neighbor 10.10.10.23 peer-group RR2-TRIADA
 neighbor 10.10.10.25 peer-group RR2-TRIADA
 neighbor 10.10.10.26 peer-group RR2-TRIADA
 neighbor 52.1.0.2 remote-as 2042
 neighbor 52.1.0.10 remote-as 301
R24#

R24#sh ip bgp summary 
BGP router identifier 24.24.24.24, local AS number 520
Neighbor        V           AS MsgRcvd MsgSent   TblVer  InQ OutQ Up/Down  State/PfxRcd
10.10.10.23     4          520      38      33        8    0    0 00:23:55        5
10.10.10.25     4          520      30      33        8    0    0 00:23:56        0
10.10.10.26     4          520      32      34        8    0    0 00:23:56        1
52.1.0.2        4         2042      37      33        8    0    0 00:23:59        1
52.1.0.10       4          301      39      34        8    0    0 00:23:59        5
R24#

R25#sh run | s r b
router bgp 520
 bgp router-id 25.25.25.25
 bgp log-neighbor-changes
 neighbor 10.10.10.23 remote-as 520
 neighbor 10.10.10.23 update-source Loopback0
 neighbor 10.10.10.23 next-hop-self
 neighbor 10.10.10.24 remote-as 520
 neighbor 10.10.10.24 update-source Loopback0
 neighbor 10.10.10.24 next-hop-self
R25#

R25#sh ip bgp summary 
Neighbor        V           AS MsgRcvd MsgSent   TblVer  InQ OutQ Up/Down  State/PfxRcd
10.10.10.23     4          520      96      79       32    0    0 01:10:22        7
10.10.10.24     4          520      34      31       32    0    0 00:24:39        7
R25

R26#sh run | s r b
router bgp 520
 bgp router-id 26.26.26.26
 bgp log-neighbor-changes
 neighbor 10.10.10.23 remote-as 520
 neighbor 10.10.10.23 next-hop-self
 neighbor 10.10.10.24 remote-as 520
 neighbor 10.10.10.24 update-source Loopback0
 neighbor 10.10.10.24 next-hop-self
 neighbor 52.1.0.6 remote-as 2042
R26#

R26#sh ip bgp summary 
BGP router identifier 26.26.26.26, local AS number 520

Neighbor        V           AS MsgRcvd MsgSent   TblVer  InQ OutQ Up/Down  State/PfxRcd
10.10.10.23     4          520      92      75       46    0    0 01:03:40        7
10.10.10.24     4          520      32      30       46    0    0 00:22:07        7
52.1.0.6        4         2042      93      93       46    0    0 01:07:45        1
R26#


```

* R15:
