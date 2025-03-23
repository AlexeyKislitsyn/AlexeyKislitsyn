
## Реализация DHCPv6 

#### Топология

![](DHCPv6.png)

#### Таблица адресации

| Устройство  | Интерфейс   | IPv6  -адрес          |
|-------------|-------------|-----------------------|
| R1          | e 0/0       | 2001:db8:acad:2::1/64 | 
| R1          | e 0/0       | fe80::1               | 
| R1          | e 0/1       | 2001:db8:acad:1::1/64 | 
| R1          | e 0/1       | fe80::1               | 
| R2          | e 0/0       | 2001:db8:acad:2::2/64 | 
| R2          | e 0/0       | fe80::2               | 
| R2          | e 0/1       | 2001:db8:acad:3::1/64 | 
| R2          | e 0/1       | fe80::1               | 
|PC-A         | NIC         | DHCP                  | 
|PC-B         | NIC         | DHCP                  | 


#### 1. Создание сети и настройка основных параметров устройства

Динамическое назначение глобальных индивидуальных IPv6-адресов можно настроить тремя способами:

•	Автоматическая конфигурация адреса без сохранения состояния (Stateless Address Autoconfiguration, SLAAC)

•	DHCPv6 без отслеживания состояния

•	Адресация DHCPv6 с учетом состояний

### 2. Настройка интерфейсов и маршрутизации для обоих маршрутизаторов.

* Настроим интерфейсы маршрутизатора R1 и R2 согласно таблицы адресации:

```
R1:

ipv6 unicast-routing
!
interface Ethernet0/0
 no ip address
 duplex auto
 ipv6 address FE80::1 link-local
 ipv6 address 2001:DB8:ACAD:2::1/64
!
interface Ethernet0/1
 no ip address
 duplex auto
 ipv6 address FE80::1 link-local
 ipv6 address 2001:DB8:ACAD:1::1/64
!
ipv6 route 2001:DB8:ACAD:3::/64 2001:DB8:ACAD:2::2

R2:

ipv6 unicast-routing
!
interface Ethernet0/0
 no ip address
 duplex auto
 ipv6 address FE80::2 link-local
 ipv6 address 2001:DB8:ACAD:2::2/64
!
interface Ethernet0/1
 no ip address
 duplex auto
 ipv6 address FE80::1 link-local
 ipv6 address 2001:DB8:ACAD:3::1/64
!
ipv6 route 2001:DB8:ACAD:1::/64 2001:DB8:ACAD:2::1
```

* Проверим ip связанность:

```
R1#ping 2001:db8:acad:3::1
Type escape sequence to abort.
Sending 5, 100-byte ICMP Echos to 2001:DB8:ACAD:3::1, timeout is 2 seconds:
!!!!!
Success rate is 100 percent (5/5), round-trip min/avg/max = 2/5/21 ms
R1#
```

#### 3. Проверка назначения адреса SLAAC от R1.


* PC-A получает адрес IPv6 с помощью метода SLAAC. Включим PC-A и убедимся, что сетевой адаптер настроен для автоматической настройки IPv6. В выводе команды show ipv6 мы увидим, что PC-A присвоил себе адрес из сети 2001:DB8:ACAD:1:/64.

```
VPCS> show ipv6    

NAME              : VPCS[1]
LINK-LOCAL SCOPE  : fe80::250:79ff:fe66:6805/64
GLOBAL SCOPE      : 2001:db8:acad:1:2050:79ff:fe66:6805/64
DNS               : 
ROUTER LINK-LAYER : aa:bb:cc:00:10:10
MAC               : 00:50:79:66:68:05
LPORT             : 20000
RHOST:PORT        : 127.0.0.1:30000
MTU:              : 1500

VPCS> 

 ```
                                    
 #### 4. Настройка и проверка сервера DHCPv6 на R1 (Stateless)
 
* Выполним настройку и проверку состояния DHCP-сервера на R1. Цель состоит в том, чтобы предоставить PC-A информацию о DNS-сервере и домене.

* Создадим пул DHCP IPv6 на R1 с именем R1-STATELESS. В составе этого пула назначим адрес DNS-сервера как 2001:db8:acad::254, а имя домена — как stateless.com.

```
ipv6 dhcp pool STATELESS
 dns-server 2001:DB8:ACAD::254
 domain-name stateless.com
!
```

* Настроим интерфейс e 0/1 на R1, чтобы предоставить флаг конфигурации OTHER для локальной сети R1 и укажим только что созданный ipv6 DHCP в качестве ресурса DHCP для этого интерфейса.

```
interface Ethernet0/1
 no ip address
 duplex auto
 ipv6 address FE80::1 link-local
 ipv6 address 2001:DB8:ACAD:1::1/64
 ipv6 nd other-config-flag
 ipv6 dhcp server STATELESS
!  
```

Таким образом, мы получили dns и домен.

#### 5. Настройка сервера DHCPv6 с сохранением состояния на R1 (Stateless)

* Создадим пул DHCPv6 на R2 для сети 2001:db8:acad:3:aaa::/80. Пул предоставит адреса локальной сети, подключенной к интерфейсу   e 0/1 на R2. В составе пула зададим DNS-сервер 2001:db8:acad::254 и задайдим доменное имя STATEFUL.com.

```
ipv6 dhcp pool R2-STATEFUL
  address prefix 2001:db8:acad:3:aaa::/80
  dns-server 2001:db8:acad::254
  domain-name STATEFUL.com

interface Ethernet0/0
  ipv6 dhcp server R2-STATEFUL
```

* Настроим HCPv6 Relay на R2

DHCPv6 Relay Agent - маршрутизатор предоставляет услуги переадресации DHCPv6, когда клиент и сервер находятся в разных сетях.

```
interface Ethernet0/1
  ipv6 nd managed-config-flag
  ipv6 dhcp relay destination 2001:db8:acad:2::1 Ethernet0/0
```

Таким образом, мы получили ipv6 адрес из заданной нами сети, dns и домен.
