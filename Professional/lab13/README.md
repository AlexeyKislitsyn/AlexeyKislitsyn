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

* Основной туннель будет проходить через R15, резервный через R14. Настроим маршрутизаторы R14 и R15 в Москве. 

Москва: В качестве destination будем использовать публичный адрес AS2042 20.0.0.254 (Санкт-Петербург), специально выделенный для этих целей.

```
R15:

interface Tunnel0
 description TUNNEL-GRE-TO-SPB
 ip address 172.31.0.1 255.255.255.252
 ip mtu 1400
 ip tcp adjust-mss 1360
 keepalive 10 1
 tunnel source Ethernet0/2
 tunnel destination 20.0.0.254
end

ip route 172.16.0.0 255.255.252.0 Tunnel0


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

Натройка R18 в СПБ:

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
