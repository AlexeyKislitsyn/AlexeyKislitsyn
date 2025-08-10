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

* Настроим маршрутизаторы R14 и R15 в Москве:
* План адресации


```
ip nat pool NAT-MSK 140.100.0.1 140.100.0.1 netmask 255.255.255.252
ip nat inside source list NAT-LAN pool NAT-MSK overload
!
ip access-list extended NAT-LAN
 permit ip 192.168.1.0 0.0.0.255 any
 permit ip 192.168.2.0 0.0.0.255 any
 deny   ip any any
!

### R15:

ip nat pool NAT-MSK 140.100.0.1 140.100.0.1 netmask 255.255.255.252
ip nat inside source list NAT-LAN pool NAT-MSK overload

ip access-list extended NAT-LAN
 permit ip 192.168.1.0 0.0.0.255 any
 permit ip 192.168.2.0 0.0.0.255 any
 deny   ip any any

```
