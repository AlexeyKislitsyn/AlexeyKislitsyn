## Настройка VPN. GRE. DmVPN

#### Цель:

1. Настроить GRE между офисами Москва и С.-Петербург
2. Настроить DMVPN между офисами Москва и Чокурдах, Лабытнанги

#### План работы

1. Настроите GRE между офисами Москва и С.-Петербург.
2. Настроите DMVMN между Москва и Чокурдах, Лабытнанги.
3. Все узлы в офисах в лабораторной работе должны иметь IP связность.



##  Схема стенда 

![](ibgp.png)

#### Выполнение.

#### GRE между офисами Москва и С.-Петербург

* Основной туннель будет проходить через R15-R18, резервный R14-R18. Настроим маршрутизаторы R14 и R15 в Москве. 

Москва: В качестве destination будем использовать публичный адрес AS2042 20.0.0.254 (Санкт-Петербург), специально выделенный для этих целей.

```
R15:

interface Tunnel1
 description TUNNEL-GRE-TO-SPB
 ip address 172.31.0.1 255.255.255.252
 ip mtu 1400
 ip tcp adjust-mss 1360
 keepalive 10 1
 tunnel source Ethernet0/2
 tunnel destination 20.0.0.254
end

ip route 172.16.0.0 255.255.252.0 Tunnel1


R14:

interface Tunnel1
 description TUNNEL-GRE-TO-SPB-REZERV
 ip address 172.31.0.5 255.255.255.252
 ip mtu 1400
 ip tcp adjust-mss 1360
 keepalive 10 1
 tunnel source Ethernet0/2
 tunnel destination 20.0.0.254
end

ip route 172.16.0.0 255.255.252.0 Tunnel1

```

Настройка R18 в СПБ:

```

interface Loopback100
 ip address 20.0.0.254 255.255.255.255
!
interface Tunnel0
 description TUNNEL-GRE-TO-MSK
 ip address 172.31.0.2 255.255.255.252
 ip mtu 1400
 ip tcp adjust-mss 1360
 keepalive 10 1
 tunnel source 20.0.0.254
 tunnel destination 30.1.0.2
!
interface Tunnel1
 description TUNNEL-GRE-TO-MSK-REZERV
 ip address 172.31.0.6 255.255.255.252
 ip mtu 1400
 ip tcp adjust-mss 1360
 keepalive 10 1
 tunnel source 20.0.0.254
 tunnel destination 101.0.0.2
!

ip route 192.168.0.0 255.255.252.0 Tunnel0
ip route 192.168.0.0 255.255.252.0 Tunnel1 250

```
* C Москвы пустим trace до СПБ: основной туннель

```
VPCS> trace 172.16.1.2  
trace to 172.16.1.2, 8 hops max, press Ctrl+C to stop
 1   192.168.2.254   3.642 ms  2.671 ms  2.730 ms
 2   10.0.1.18   4.536 ms  3.712 ms  3.709 ms
 3   10.0.0.14   5.239 ms  5.889 ms  5.288 ms
 4   172.31.0.2   9.010 ms  8.586 ms  9.455 ms
 5   10.1.1.1   10.224 ms  10.698 ms  8.000 ms
 6   10.1.0.9   8.401 ms  10.068 ms  10.037 ms
 7   *172.16.1.2   11.401 ms (ICMP type:3, code:3, Destination port unreachable)

VPCS>
```

* При отказе R15 трафик идет через резервный туннель на R14:

```
VPCS> trace 172.16.1.2 
trace to 172.16.1.2, 8 hops max, press Ctrl+C to stop
 1   192.168.2.254   2.048 ms  2.065 ms  1.672 ms
 2   10.0.1.18   2.570 ms  2.299 ms  2.574 ms
 3   10.0.0.10   3.299 ms  2.987 ms  3.093 ms
 4   172.31.0.6   7.739 ms  6.196 ms  5.468 ms
 5   10.1.1.1   6.317 ms  6.341 ms  5.664 ms
 6   10.1.0.9   5.097 ms  5.221 ms  5.498 ms
 7   *172.16.1.2   6.636 ms (ICMP type:3, code:3, Destination port unreachable)

VPCS>
```

Таким образом, схема отрабатывает корректно.
 
#### DMVPN между офисами Москва и Чокурдах, Лабытнанги

* Построим DMVPN между офисами Москва, Чокурдах и Лабытнанги. В Москве R14, R15  выступают в роли  HUB, в Чокурдах, Лабытнанги - SPOKE. Кроме того, SPOKE могут напрямую общаться между собой минуя HUB. В кчестве протокола маршрутизации будем использовать ibgp. R14 является основным хабом. В случае выхода из строя основного хаба, либо недоступности удаленных офисов через провайдера Ламас - трафик будет проходить через резервный хаб R14.

Офис в Москве - HUB:

```
R14 overlay: 172.30.0.129/25
R15 overlay: 172.30.0.1/25
LAN : 192.168.0.0/17
```

Лабынтаги - SPOKE:

```
R27 overlay: 172.30.0.2/25
R27 overlay: 172.30.0.130/25 Rezerv to R14
LAN: 100.64.0.0/24
```

Чокурдах - SPOKE:

```
R28 overlay: 172.30.0.3/25
R28 overlay: 172.30.0.131/25 Rezerv to R14
LAN: 172.16.30.0/23
```

* Настроим туннельные интерфейсы в Москве:

R15 - HUB: 

```
interface Tunnel1001
 description DMVPN
 ip address 172.30.0.1 255.255.255.128
 no ip redirects
 ip mtu 1400
 ip nhrp network-id 200
 ip tcp adjust-mss 1360
 keepalive 10 1
 tunnel source Ethernet0/2
 tunnel mode gre multipoint
 tunnel key 200
!         

```

R14 - HUB:

```
interface Tunnel1002
 description DMVPN-REZERV
 ip address 172.30.0.129 255.255.255.128
 no ip redirects
 ip mtu 1400
 ip nhrp network-id 201
 ip tcp adjust-mss 1360
 keepalive 10 1
 tunnel source Ethernet0/2
 tunnel mode gre multipoint
 tunnel key 201
!
```

Лабынтаги - SPOKE:

```
interface Tunnel1001
 description DMVPN-TO-MSK-R15
 ip address 172.30.0.2 255.255.255.128
 no ip redirects
 ip mtu 1400
 ip nhrp map multicast 30.1.0.2
 ip nhrp map 172.30.0.1 30.1.0.2
 ip nhrp network-id 200
 ip nhrp nhs 172.30.0.1
 ip tcp adjust-mss 1360
 keepalive 10 1
 tunnel source Ethernet0/0
 tunnel mode gre multipoint
 tunnel key 200
!
interface Tunnel1002
 description DMVPN-TO-MSK-R14-REZERV
 ip address 172.30.0.130 255.255.255.128
 no ip redirects
 ip mtu 1400
 ip nhrp map multicast 101.0.0.2
 ip nhrp map 172.30.0.129 101.0.0.2
 ip nhrp network-id 201
 ip nhrp nhs 172.30.0.129
 ip tcp adjust-mss 1360
 keepalive 10 1
 tunnel source Ethernet0/0
 tunnel mode gre multipoint
 tunnel key 201
!         
```

Чокурдах - SPOKE:

```
interface Tunnel1001
 description DMVPN-TO-MSK-R15
 ip address 172.30.0.3 255.255.255.128
 no ip redirects
 ip mtu 1400
 ip nhrp map multicast 30.1.0.2
 ip nhrp map 172.30.0.1 30.1.0.2
 ip nhrp network-id 200
 ip nhrp nhs 172.30.0.1
 ip tcp adjust-mss 1360
 keepalive 10 1
 tunnel source Ethernet0/0
 tunnel mode gre multipoint
 tunnel key 200
!         
interface Tunnel1002
 description DMVPN-TO-MSK-R14-REZERV
 ip address 172.30.0.131 255.255.255.128
 no ip redirects
 ip mtu 1400
 ip nhrp map multicast 101.0.0.2
 ip nhrp map 172.30.0.129 101.0.0.2
 ip nhrp network-id 201
 ip nhrp nhs 172.30.0.129
 ip tcp adjust-mss 1360
 keepalive 10 1
 tunnel source Ethernet0/1
 tunnel mode gre multipoint
 tunnel key 201
! 
```

Хаб R14 и R15 динамически изучили споки, после чего прошла их регистрация на хабе:

```
R15#sh dmvpn 
Interface: Tunnel1001, IPv4 NHRP Details 
Type:Hub, NHRP Peers:2, 

  Ent  Peer NBMA Addr Peer Tunnel Add State  UpDn Tm Attrb
 ----- --------------- --------------- ----- -------- -----
     1 52.1.1.2             172.30.0.2    UP 01:07:16     D
     1 52.1.2.10            172.30.0.3    UP 01:07:31     D

R15#

R14#sh dmvpn 
Interface: Tunnel1002, IPv4 NHRP Details 
Type:Hub, NHRP Peers:2, 

  Ent  Peer NBMA Addr Peer Tunnel Add State  UpDn Tm Attrb
 ----- --------------- --------------- ----- -------- -----
     1 52.1.1.2           172.30.0.130    UP 01:05:39     D
     1 52.1.1.10          172.30.0.131    UP 01:05:54     D

R14#
```
