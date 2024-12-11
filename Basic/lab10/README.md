# Настройка протокола OSPFv2 для одной области

### Топология

![](1.png)

### Таблица адресации

| Устройство  | Интерфейс   | IP  -адрес          | Маска подсети  | 
|-------------|-------------|---------------------|----------------|
| R1          | e0/0        | 10.53.0.1           | 255.255.255.0  |
| R1          | Loopback 1  | 172.16.1.1          | 255.255.255.0  | 
| R2          | e0/0        | 10.53.0.2           | 255.255.255.0  |        
| R2          | Loopback 1  | 192.168.1.1         | 255.255.255.0  | 


### 1. Настройка и проверка базовой работы протокола  OSPFv2 для одной области

* Выполним базовую настройку маршрутизаторов и назначим ip адреса на интерфейсах согласно таблицы адресации:

```
R1#sh ip int brief 
Interface                  IP-Address      OK? Method Status                Protocol
Ethernet0/0                10.53.0.1       YES manual up                    up      
Ethernet0/1                unassigned      YES unset  administratively down down    
Ethernet0/2                unassigned      YES unset  administratively down down    
Ethernet0/3                unassigned      YES unset  administratively down down    
Loopback1                  172.16.1.1      YES manual up                    up      
R1#

R2#sh ip int brief 
Interface                  IP-Address      OK? Method Status                Protocol
Ethernet0/0                10.53.0.2       YES manual up                    up      
Ethernet0/1                unassigned      YES unset  administratively down down    
Ethernet0/2                unassigned      YES unset  administratively down down    
Ethernet0/3                unassigned      YES unset  administratively down down    
Loopback1                  192.168.1.1     YES manual up                    up      
R2#
```

* Настроим процесс OSPF на каждом маршрутизаторе, выбрав идентификатор процесса равным 56. Зададим router-id для протокола OSPF на каждом маршрутизаторе:

```
R1(config)#router ospf 56
R1(config-router)#router-id 1.1.1.1

R2(config)#router ospf 56
R2(config-router)#router-id 2.2.2.2
```

* На обоих маршрутизаторах добавим интерфейсы е 0/0 в процесс OSPF 56 area 0:

```
R1(config)#int e 0/0
R1(config-if)#ip ospf 56 area 0

R2(config)#int e 0/0
R2(config-if)#ip ospf 56 area 0
```

* Так же, на R2 добавим интерфейс Loopback1 в процесс OSPF 56 area 0:

```
R2(config)#int loop 1
R2(config-if)#ip ospf 56 area 0
```

* Проверим правильность добавления интерфейсов, участвующих в процессе ospf:

```
R1#sh ip ospf interface brief 
Interface    PID   Area            IP Address/Mask    Cost  State Nbrs F/C
Et0/0        56    0               10.53.0.1/24       10    BDR   1/1
R1#

R2#sh ip ospf interface brief 
Interface    PID   Area            IP Address/Mask    Cost  State Nbrs F/C
Lo1          56    0               192.168.1.1/24     1     LOOP  0/0
Et0/0        56    0               10.53.0.2/24       10    DR    1/1
R2#
```

* Убедимся, что R1 и R2 установили смежность по протоколу ospf:

```
R1#sh ip ospf neighbor 

Neighbor ID     Pri   State           Dead Time   Address         Interface
2.2.2.2           1   FULL/DR         00:00:31    10.53.0.2       Ethernet0/0
R1#

R2#sh ip ospf neighbor 

Neighbor ID     Pri   State           Dead Time   Address         Interface
1.1.1.1           1   FULL/BDR        00:00:34    10.53.0.1       Ethernet0/0
R2#
```

Т.к у R2 значение идентификатора маршрутизатора (router-id) больше, чем у маршрутизатора R1, следовательно маршрутизатор R2 стал назначенным маршрутизатором (DR). Данный критерий является вторым при выборе DR и BDR. Первостепенным является приоритет, который по умолчанию равен 1 для обоих маршрутизаторов.

* На R1 выполним команду show ip route ospf, чтобы убедиться, что сеть R2 Loopback 1 присутствует в таблице маршрутизации:

```
R1#sh ip route ospf
Gateway of last resort is not set

      192.168.1.0/32 is subnetted, 1 subnets
O        192.168.1.1 [110/11] via 10.53.0.2, 00:15:14, Ethernet0/0
R1#
```

* Проверим командой Ping доступность адреса интерфейса R2 Loopback 1 из R1:

```
R1#ping 192.168.1.1  
Type escape sequence to abort.
Sending 5, 100-byte ICMP Echos to 192.168.1.1, timeout is 2 seconds:
!!!!!
Success rate is 100 percent (5/5), round-trip min/avg/max = 4/5/8 ms
R1#
```

### 2. Оптимизация и проверка конфигурации OSPFv2 для одной области

* На R1 настроим приоритет OSPF интерфейса e 0/0 на 50 и убедимся, что R1 является назначенным маршрутизатором:

```
R1(config)#int e0/0 
R1(config-if)#ip ospf priority 50
R2#clear ip ospf process

R2#sh ip ospf neighbor 

Neighbor ID     Pri   State           Dead Time   Address         Interface
1.1.1.1          50   FULL/DR         00:00:33    10.53.0.1       Ethernet0/0
R2#
```

* На обоих маршрутизаторах поменяем интервал сообщений приветствия на значение 30 сек (по умолччанию Hello 10, Dead 40):

```
R1(config)#int e 0/0
R1(config-if)#ip ospf hello-interval 30
```
Автоматически меняется Dead интервал на 120 сек:

```
R1(config-if)#do sh ip osp int e 0/0
Timer intervals configured, Hello 30, Dead 120, Wait 120, Retransmit 5
```
* Настроим на маршрутизаторе R1 маршрутом по умолчанию интерфейс Loopback1 и включим распространение данного маршрута протоколом OSPF:

```
1(config)#ip route 0.0.0.0 0.0.0.0 loop 1
%Default route without gateway, if not a point-to-point interface, may impact performance
R1(config)#router ospf 56
R1(config-router)#default-information originate 
```

Посмотрим наличие маршрута по умолччанию на R2:

```
R2#sh ip route 
Gateway of last resort is 10.53.0.1 to network 0.0.0.0

O*E2  0.0.0.0/0 [110/1] via 10.53.0.1, 00:13:42, Ethernet0/0
      10.0.0.0/8 is variably subnetted, 2 subnets, 2 masks
C        10.53.0.0/24 is directly connected, Ethernet0/0
L        10.53.0.2/32 is directly connected, Ethernet0/0
      192.168.1.0/24 is variably subnetted, 2 subnets, 2 masks
C        192.168.1.0/24 is directly connected, Loopback1
L        192.168.1.1/32 is directly connected, Loopback1
R2#
```

* Для маршрутизатора R2 переведем интерфейс Loopback1 в режим point-to-point:

```
R2(config)#int loop 1
R2(config-if)#ip ospf network point-to-point 

R2#sh ip ospf int brief 
Interface    PID   Area            IP Address/Mask    Cost  State Nbrs F/C
Lo1          56    0               192.168.1.1/24     1     P2P   0/0
Et0/0        56    0               10.53.0.2/24       10    BDR   1/1
R2#
```

также запретим отправлять объявления OSPF в сеть Loopback1:

```
R2(config)#router ospf 56
R2(config-router)#passive-interface loopback 1

R2#sh ip protocols | begin ospf
Routing Protocol is "ospf 56"
  Outgoing update filter list for all interfaces is not set
  Incoming update filter list for all interfaces is not set
  Router ID 2.2.2.2
  Number of areas in this router is 1. 1 normal 0 stub 0 nssa
  Maximum path: 4
  Routing for Networks:
  Routing on Interfaces Configured Explicitly (Area 0):
    Loopback1
    Ethernet0/0
  Passive Interface(s):
    Loopback1
  Routing Information Sources:
    Gateway         Distance      Last Update
    1.1.1.1              110      00:44:13
  Distance: (default is 110)

R2#
```

* Изменим базовую пропускную способность для маршрутизаторов. После этой настройки перезапустим OSPF с помощью команды clear ip ospf process:

Выведем текущую конфигурацию:

```
R1#sh ip ospf | include band
 Reference bandwidth unit is 100 mbps
R1#

R1#sh int e 0/0 | include BW
  MTU 1500 bytes, BW 10000 Kbit/sec, DLY 1000 usec, 
R1#

R1#sh ip ospf interface brief 
Interface    PID   Area            IP Address/Mask    Cost  State Nbrs F/C
Et0/0        56    0               10.53.0.1/24       10    DR    1/1
R1#
```

Стоимость (cost) рассчитывается по формуле cost = reference bandwidth / bandwidth интерфейса. Таким образом, по умолчанию cost=100/10=10 mbps, что мы и видем в выводе команды R1#sh ip ospf interface brief. Изменим Reference bandwidth на 10 mbps, в этом случае cost у нас получится 1.

```
R1(config)#router ospf 56
R1(config-router)#auto-cost reference-bandwidth 10
R1#lear ip ospf process 

R2(config)#router ospf 56
R2(config-router)#auto-cost reference-bandwidth 10
R2#clear ip ospf process 
```
Проверим изменение стоимости на маршрутизаторах (cost=1) и Reference bandwidth:

```
R1#sh ip ospf interface brief 
Interface    PID   Area            IP Address/Mask    Cost  State Nbrs F/C
Et0/0        56    0               10.53.0.1/24       1     DR    1/1
R1#sh ip ospf | include band
 Reference bandwidth unit is 10 mbps
R1#


R2#sh ip ospf interface brief 
Interface    PID   Area            IP Address/Mask    Cost  State Nbrs F/C
Lo1          56    0               192.168.1.1/24     1     P2P   0/0
Et0/0        56    0               10.53.0.2/24       1     BDR   1/1
R2#sh ip ospf | include band
 Reference bandwidth unit is 10 mbps
R1#
```
* Проверим итоговое состояние таблицы маршрутизации: 

```
R1# sh ip route
Gateway of last resort is 0.0.0.0 to network 0.0.0.0

S*    0.0.0.0/0 is directly connected, Loopback1
      10.0.0.0/8 is variably subnetted, 2 subnets, 2 masks
C        10.53.0.0/24 is directly connected, Ethernet0/0
L        10.53.0.1/32 is directly connected, Ethernet0/0
      172.16.0.0/16 is variably subnetted, 2 subnets, 2 masks
C        172.16.1.0/24 is directly connected, Loopback1
L        172.16.1.1/32 is directly connected, Loopback1
O     192.168.1.0/24 [110/2] via 10.53.0.2, 01:45:58, Ethernet0/0
R1#

Gateway of last resort is 10.53.0.1 to network 0.0.0.0

R1# sh ip route
O*E2  0.0.0.0/0 [110/1] via 10.53.0.1, 01:46:48, Ethernet0/0
      10.0.0.0/8 is variably subnetted, 2 subnets, 2 masks
C        10.53.0.0/24 is directly connected, Ethernet0/0
L        10.53.0.2/32 is directly connected, Ethernet0/0
      192.168.1.0/24 is variably subnetted, 2 subnets, 2 masks
C        192.168.1.0/24 is directly connected, Loopback1
L        192.168.1.1/32 is directly connected, Loopback1
R2#
```

Проверим доступность внешней сети на R1 (имитация внешней сети на инт loopback 1):

```
R2#ping 172.16.1.1
Type escape sequence to abort.
Sending 5, 100-byte ICMP Echos to 172.16.1.1, timeout is 2 seconds:
!!!!!
Success rate is 100 percent (5/5), round-trip min/avg/max = 1/3/5 ms
R2#
```

В данном случае, стоимость маршрута до сети 192.168.1.0/24 складывается из 2 составляющих: канал между маршрутизаторами со стоимостью 1 и выходной Loopback1 со стоимостью 1 (1 по умолчанию для интерфейсов loopback), в сумме получается 2. В случае же с маршрутом по умолчанию для маршрутизатора R2, трафику нужно лишь пройти до интерфейса 10.53.0.1, до которого только один канал со стоимостью 1.
