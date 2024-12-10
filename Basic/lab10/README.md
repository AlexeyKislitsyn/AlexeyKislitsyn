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


### 1 . Настройка и проверка базовой работы протокола  OSPFv2 для одной области

Выполним базовую настройку маршрутизаторов и назначим ip адреса на интерфейсах согласно таблицы адресации:

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

Настроим процесс OSPF на каждом маршрутизаторе, выбрав идентификатор процесса равным 56. Зададим идентификаторы маршрутизатора для протокола OSPF на каждом маршрутизаторе:

R1(config)#router ospf 56
R1(config-router)#router-id 1.1.1.1

R2(config)#router ospf 56
R2(config-router)#router-id 2.2.2.2

На обоих маршрутизаторах зададим сеть, которая будет участвовать в работе протокола OSPF:

R1(config)#int e 0/0
R1(config-if)#ip ospf 56 area 0

R2(config)#int e 0/0
R2(config-if)#ip ospf 56 area 0

Так же на R2 добавим интерфейс Loopback1 в процесс ospf:

R2(config)#int loop 1
R2(config-if)#ip ospf 56 area 0


### Шаг 1. Настройка коммутаторов S1 и S2.
