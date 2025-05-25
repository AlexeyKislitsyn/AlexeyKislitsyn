## IS-IS

#### Цель:Настроить IS-IS в офисе Триада

1. Настроите IS-IS в ISP Триада.
2. R23 и R25 находятся в зоне 2222.
3. R24 находится в зоне 24.
4. R26 находится в зоне 26.

![](Moskow1.png)

#### Выполнение.

* Выполним настройку протокола IS-IS на маршрутизаторах R23,R24,R25,R26 согласно тоологии сети:

* R23:

```
!
interface Loopback0
 ip address 10.10.10.23 255.255.255.255
 ip router isis 
 isis circuit-type level-2-only
!
interface Ethernet0/1
 ip address 10.0.0.1 255.255.255.252
 ip router isis 
 isis circuit-type level-1
!
interface Ethernet0/2
 ip address 10.0.0.5 255.255.255.252
 ip router isis 
 isis circuit-type level-2-only
!
router isis
 net 49.2222.0000.0000.0023.00
!
```

* R24:

```
!
interface Loopback0
 ip address 10.10.10.24 255.255.255.255
 ip router isis 
 isis circuit-type level-2-only
!
interface Ethernet0/1
 ip address 10.0.0.9 255.255.255.252
 ip router isis 
 isis circuit-type level-2-only
!
interface Ethernet0/2
 ip address 10.0.0.6 255.255.255.252
 ip router isis 
 isis circuit-type level-2-only
!
router isis
 net 49.0024.0000.0000.0024.00
!
```

* R25:

```
interface Loopback0
 ip address 10.10.10.25 255.255.255.255
 ip router isis 
 isis circuit-type level-2-only
!
interface Ethernet0/0
 ip address 10.0.0.2 255.255.255.252
 ip router isis 
 isis circuit-type level-1
!
interface Ethernet0/2
 ip address 10.0.0.14 255.255.255.252
 ip router isis 
 isis circuit-type level-2-only
!
router isis
 net 49.2222.0000.0000.0025.00
!
```

* R26:

```
!
interface Loopback0
 ip address 10.10.10.26 255.255.255.255
 ip router isis 
 isis circuit-type level-2-only
!
interface Ethernet0/0
 ip address 10.0.0.10 255.255.255.252
 ip router isis 
 isis circuit-type level-2-only
!
interface Ethernet0/2
 ip address 10.0.0.13 255.255.255.252
 ip router isis 
 isis circuit-type level-2-only
!
router isis
 net 49.0026.0000.0000.0026.00
!

```

* На примере R23 посмотрим установление отношений соседства, топологию и таблицу маршрутизации :

```
R23#sh isis neighbors 

System Id      Type Interface   IP Address      State Holdtime Circuit Id
R24            L2   Et0/2       10.0.0.6        UP    9        R24.02             
R25            L1   Et0/1       10.0.0.2        UP    7        R25.01             
R23#

R23#sh isis topology 

IS-IS TID 0 paths to level-1 routers
System Id            Metric     Next-Hop             Interface   SNPA
R23                  --
R25                  10         R25                  Et0/1       aabb.cc01.9000 

IS-IS TID 0 paths to level-2 routers
System Id            Metric     Next-Hop             Interface   SNPA
R23                  --
R24                  10         R24                  Et0/2       aabb.cc01.8020 
R25                  30         R24                  Et0/2       aabb.cc01.8020 
R26                  20         R24                  Et0/2       aabb.cc01.8020 
R23#


R23#sh ip route 

      10.0.0.0/8 is variably subnetted, 10 subnets, 2 masks
C        10.0.0.0/30 is directly connected, Ethernet0/1
L        10.0.0.1/32 is directly connected, Ethernet0/1
C        10.0.0.4/30 is directly connected, Ethernet0/2
L        10.0.0.5/32 is directly connected, Ethernet0/2
i L2     10.0.0.8/30 [115/20] via 10.0.0.6, 01:07:20, Ethernet0/2
i L2     10.0.0.12/30 [115/30] via 10.0.0.6, 00:56:05, Ethernet0/2
C        10.10.10.23/32 is directly connected, Loopback0
i L2     10.10.10.24/32 [115/20] via 10.0.0.6, 01:07:20, Ethernet0/2
i L2     10.10.10.25/32 [115/40] via 10.0.0.6, 00:53:42, Ethernet0/2
i L2     10.10.10.26/32 [115/30] via 10.0.0.6, 00:56:05, Ethernet0/2
      101.0.0.0/8 is variably subnetted, 2 subnets, 2 masks
C        101.0.0.0/30 is directly connected, Ethernet0/0
L        101.0.0.1/32 is directly connected, Ethernet0/0
R23#

```


