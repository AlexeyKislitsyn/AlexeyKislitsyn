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

Выполним базовую настройку маршрутизаторов и назначим ip адреса на интерфейсах согласно таблицы адресации:

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

Настроим процесс OSPF на каждом маршрутизаторе, выбрав идентификатор процесса равным 56. Зададим идентификаторы маршрутизатора для протокола OSPF на каждом маршрутизаторе:

```
R1(config)#router ospf 56
R1(config-router)#router-id 1.1.1.1

R2(config)#router ospf 56
R2(config-router)#router-id 2.2.2.2
```

На обоих маршрутизаторах добавим интерфейсы е 0/0 в процесс OSPF 56 area 0:

```
R1(config)#int e 0/0
R1(config-if)#ip ospf 56 area 0

R2(config)#int e 0/0
R2(config-if)#ip ospf 56 area 0
```

Так же, на R2 добавим интерфейс Loopback1 в процесс OSPF 56 area 0:

```
R2(config)#int loop 1
R2(config-if)#ip ospf 56 area 0
```

Проверим правильность добавления интерфейсов, уччаствущих в процессе ospf:

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

Убедимся, что R1 и R2 установили смежность по протоколу ospf:

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

На R1 выполним команду show ip route ospf, чтобы убедиться, что сеть R2 Loopback1 присутствует в таблице маршрутизации:

```
R1#sh ip route ospf
Gateway of last resort is not set

      192.168.1.0/32 is subnetted, 1 subnets
O        192.168.1.1 [110/11] via 10.53.0.2, 00:15:14, Ethernet0/0
R1#
```

Проверим командой Ping доступность адреса интерфейса R2 Loopback 1 из R1:

```
R1#ping 192.168.1.1  
Type escape sequence to abort.
Sending 5, 100-byte ICMP Echos to 192.168.1.1, timeout is 2 seconds:
!!!!!
Success rate is 100 percent (5/5), round-trip min/avg/max = 4/5/8 ms
R1#
```

### 2. Оптимизация и проверка конфигурации OSPFv2 для одной области

На R1 настроим приоритет OSPF интерфейса e 0/0 на 50 и убедимся, что R1 является назначенным маршрутизатором:

```
R1(config)#int e0/0 
R1(config-if)#ip ospf priority 50
R1(config-if)#

R2#clear ip ospf process
R2#sh ip ospf neighbor 

Neighbor ID     Pri   State           Dead Time   Address         Interface
1.1.1.1          50   FULL/DR         00:00:33    10.53.0.1       Ethernet0/0
R2#
```

На обоих маршрутизаторах поменяем интервал сообщений приветствия на значение 30 сек (по умолччанию Hello 10, Dead 40):

```
R1(config)#int e 0/0
R1(config-if)#ip ospf hello-interval 30
```

автоматически меняется Dead интервал на 120 сек (1 к 4):

```
R1(config-if)#do sh ip osp int e 0/0
Timer intervals configured, Hello 30, Dead 120, Wait 120, Retransmit 5
```



