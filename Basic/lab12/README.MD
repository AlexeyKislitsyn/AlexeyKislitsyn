# Настройка NAT для IPv4

### Топология

![](1.png)

### Таблица адресации

| Устройство  | Интерфейс    | IP  -адрес         | Маска подсети   |
|-------------|--------------|--------------------|-----------------|
| R1          | e 0/0        | 209.165.200.230    | 255.255.255.248 | 
| R1          | e 0/1        | 192.168.1.1        | 255.255.255.0   | 
| R2          | e 0/0        | 209.165.200.225    | 255.255.255.248 | 
| R2          | Lo 1         | 209.165.200.1      | 255.255.255.224 | 
| S1          | VLAN 1       | 192.168.1.11       | 255.255.255.0   | 
| S2          | VLAN 1       | 192.168.1.12       | 255.255.255.0   | 
| PC-A        | NIC          | 192.168.1.2        | 255.255.255.0   | 
| PC-B        | NIC          | 192.168.1.3        | 255.255.255.0   | 

Интернет-провайдер выделил компании общедоступное пространство IP-адресов 209.165.200.224/29. Эта сеть используется для обращения к каналу между маршрутизатором ISP (R2) и шлюзом компании (R1). Первый адрес (209.165.200.225) назначается интерфейсу e 0/0 на R2, а последний адрес (209.165.200.230) назначается интерфейсу e 0/0 на R1. Остальные адреса (209.165.200.226-209.165.200.229) будут использоваться для предоставления доступа в Интернет хостам компании. Маршрут по умолчанию используется от R1 до R2. Подключение интернет-провайдера к Интернету смоделировано loopback-адресом на маршрутизаторе интернет-провайдера.

#### Задачи

1. Создание сети и настройка основных параметров устройства
2. Настройка и проверка NAT для IPv4
3. Настройка и проверка PAT для IPv4
4. Настройка и проверка статического NAT для IPv4

---

### 1. Создание сети и настройка основных параметров устройства

* Создана топология сети в EVE-NG.
* Выполним настройку интерфейсов активного оборудования согласно талицы ip адресации.

```
S1#sh ip int brief 
Interface              IP-Address      OK? Method Status                Protocol
Ethernet0/0            unassigned      YES unset  up                    up      
Ethernet0/1            unassigned      YES unset  up                    up      
Ethernet0/2            unassigned      YES unset  up                    up      
Ethernet0/3            unassigned      YES unset  up                    up      
Vlan1                  192.168.1.11    YES manual up                    up      
S1#
S1#sh ip redirects 
Default gateway is 192.168.1.1


S2#sh ip int brief 
Interface              IP-Address      OK? Method Status                Protocol
Ethernet0/0            unassigned      YES unset  up                    up      
Ethernet0/1            unassigned      YES unset  up                    up      
Ethernet0/2            unassigned      YES unset  up                    up      
Ethernet0/3            unassigned      YES unset  up                    up      
Vlan1                  192.168.1.12    YES manual up                    up      
S2#
S2#sh ip redirects 
Default gateway is 192.168.1.1

R1#sh ip int brief 
Interface                  IP-Address      OK? Method Status                Protocol
Ethernet0/0                209.165.200.230 YES manual up                    up      
Ethernet0/1                192.168.1.1     YES manual up                    up      
Ethernet0/2                unassigned      YES unset  administratively down down    
Ethernet0/3                unassigned      YES unset  administratively down down    
R1#

R1#sh ip route static 
S*    0.0.0.0/0 [1/0] via 209.165.200.225

R2#sh ip int brief 
Interface                  IP-Address      OK? Method Status                Protocol
Ethernet0/0                209.165.200.225 YES manual up                    up      
Ethernet0/1                unassigned      YES unset  administratively down down    
Ethernet0/2                unassigned      YES unset  administratively down down    
Ethernet0/3                unassigned      YES unset  administratively down down    
Loopback1                  209.165.200.1   YES manual up                    up      
R2#

```

### 2. Настройка и проверка NAT для IPv4

* Настроим NAT на R1, используя пул из трех адресов 209.165.200.226-209.165.200.228.

1. Настроим список доступа, определяющий какие адреса будут транслироваться. В данном случае все устройства в локальной сети R1 имеют право на трансляцию.

```
R1(config)#ip access-list standard LAN_TO_NAT
R1(config-std-nacl)#permit 192.168.1.0 0.0.0.255
```

2. Создадим пул NAT адресов:

```
R1(config)# ip nat pool PUBLIC_ACCESS 209.165.200.226 209.165.200.228 netmask 255.255.255.248
```

3. Настроим трансляцию адресов, связав ранее созданные ACL и пул адресов с процессом преобразования адресов:

```
R1(config)#ip nat inside source list LAN_TO_NAT pool PUBLIC_ACCESS 
```

4. Зададим внутренний и внешний интерфейсы:

```
R1(config)# int e0/1
R1(config-if)# ip nat inside
R1(config-if)# int e0/0
R1(config-if)# ip nat outside
```

* Проведем некоторые тесты:

1. С PC-B выполним PING до интерфейса Lo1 (209.165.200.1) на R2 и отобразим NAT-таблицу на R1:

```
R1#sh ip nat translations 
Pro Inside global      Inside local       Outside local      Outside global
icmp 209.165.200.226:29901 192.168.1.3:29901 209.165.200.1:29901 209.165.200.1:29901
icmp 209.165.200.226:30413 192.168.1.3:30413 209.165.200.1:30413 209.165.200.1:30413
icmp 209.165.200.226:30669 192.168.1.3:30669 209.165.200.1:30669 209.165.200.1:30669
icmp 209.165.200.226:30925 192.168.1.3:30925 209.165.200.1:30925 209.165.200.1:30925
icmp 209.165.200.226:31181 192.168.1.3:31181 209.165.200.1:31181 209.165.200.1:31181
--- 209.165.200.226    192.168.1.3        ---                ---
R1# 

```

2. С PC-A выполним PING до интерфейса Lo1 (209.165.200.1) на R2 и отобразим NAT-таблицу на R1:

```
R1#sh ip nat translations 
Pro Inside global      Inside local       Outside local      Outside global
icmp 209.165.200.227:37326 192.168.1.2:37326 209.165.200.1:37326 209.165.200.1:37326
icmp 209.165.200.227:37582 192.168.1.2:37582 209.165.200.1:37582 209.165.200.1:37582
icmp 209.165.200.227:37838 192.168.1.2:37838 209.165.200.1:37838 209.165.200.1:37838
icmp 209.165.200.227:38094 192.168.1.2:38094 209.165.200.1:38094 209.165.200.1:38094
icmp 209.165.200.227:38350 192.168.1.2:38350 209.165.200.1:38350 209.165.200.1:38350
--- 209.165.200.227    192.168.1.2        ---                ---
icmp 209.165.200.226:40398 192.168.1.3:40398 209.165.200.1:40398 209.165.200.1:40398
icmp 209.165.200.226:40654 192.168.1.3:40654 209.165.200.1:40654 209.165.200.1:40654
icmp 209.165.200.226:40910 192.168.1.3:40910 209.165.200.1:40910 209.165.200.1:40910
icmp 209.165.200.226:41166 192.168.1.3:41166 209.165.200.1:41166 209.165.200.1:41166
icmp 209.165.200.226:41422 192.168.1.3:41422 209.165.200.1:41422 209.165.200.1:41422
Pro Inside global      Inside local       Outside local      Outside global
--- 209.165.200.226    192.168.1.3        ---                ---
R1#
```

Таким образом, произошла динамическая трансляция адресов из внутренней сети во внешний пул адресов, и из внешнего пула NAT выбирается следующий свободный адрес.

```
--- 209.165.200.227    192.168.1.2        ---                ---
--- 209.165.200.226    192.168.1.3        ---                ---
```

3. Выполним команду PING на коммутаторах S1 и S2:

```
R1#sh ip nat translations 
Pro Inside global      Inside local       Outside local      Outside global
--- 209.165.200.226    192.168.1.2        ---                ---
--- 209.165.200.228    192.168.1.3        ---                ---
--- 209.165.200.227    192.168.1.11       ---                ---
R1#

```

Учитывая, что пул ограничен тремя адресами, PING на коммутаторе S2  завершается неудачей:

```
S2#ping 209.165.200.1
Type escape sequence to abort.
Sending 5, 100-byte ICMP Echos to 209.165.200.1, timeout is 2 seconds:
U.U.U
Success rate is 0 percent (0/5)
S2#
```

С помощью следующей команды можно увидеть время резервирования записей для выполненных трансляций, 24 часа:

```
R1#sh ip nat translations verbose 
Pro Inside global      Inside local       Outside local      Outside global
--- 209.165.200.227    192.168.1.2        ---                ---
    create 00:02:04, use 00:01:59 timeout:86400000, left 23:58:00, Map-Id(In): 1, 
    flags: 
none, use_count: 0, entry-id: 7, lc_entries: 0
--- 209.165.200.226    192.168.1.3        ---                ---
    create 00:02:46, use 00:02:41 timeout:86400000, left 23:57:18, Map-Id(In): 1, 
    flags: 
none, use_count: 0, entry-id: 1, lc_entries: 0
--- 209.165.200.228    192.168.1.11       ---                ---
    create 00:01:41, use 00:01:41 timeout:86400000, left 23:58:18, Map-Id(In): 1, 
    flags: 
none, use_count: 0, entry-id: 13, lc_entries: 0
R1#
```






