## BGP. Фильтрация

#### Цель:

Настроить фильтрацию для офиса Москва

Настроить фильтрацию для офиса С.-Петербург

#### План раоты

1. Настроить фильтрацию в офисе Москва так, чтобы не появилось транзитного трафика(As-path).
2. Настроить фильтрацию в офисе С.-Петербург так, чтобы не появилось транзитного трафика(Prefix-list).
3. Настроить провайдера Киторн так, чтобы в офис Москва отдавался только маршрут по умолчанию.
4. Настроить провайдера Ламас так, чтобы в офис Москва отдавался только маршрут по умолчанию и префикс офиса С.-Петербург.
5. Все сети в лабораторной работе должны иметь IP связность.



##  Схема стенда 

![](ibgp.png)

#### Выполнение.

#### Настройка фильтрацию в офисе Москва так, чтобы не появилось транзитного трафика.

На данный момент наша AS не является транзитной. Но все равно На R14 и R15 напишем правила для избежания транзитного трафика в будущем (возможны изменения структуры сети ит.д). Настроим фильтрацияю по as-path, т.е соседу мы будет отдавать только те сети, которые сгенерированы в нашей AS. Привяжем фильтр на R14 через filter-list, на R15 -  route-map. 

  
```
### R14:
ip as-path access-list 10 permit ^$
ip as-path access-list 10 deny .*

R14#sh run | s r b
router bgp 1001
 bgp router-id 14.14.14.14
 neighbor 101.0.0.1 filter-list 10 out
R14#


### R15:

ip as-path access-list 10 permit ^$
ip as-path access-list 10 deny .*
!
route-map rm_LOCAL_ONLY_OUT permit 10
 match as-path 10
! 
router bgp 1001
 bgp router-id 15.15.15.15
 neighbor 30.1.0.1 route-map rm_LOCAL_ONLY_OUT out
!
```

#### Настроим фильтрацию в офисе С.-Петербург так, чтобы не появилось транзитного трафика

В Санкт-Петербурге настроим фильтрация транзитного трафика на основе prefix-list:

Разрешим анонсировать только наши сети:

```
ip prefix-list pl_LOCAL_ONLY_OUT seq 10 permit 20.0.0.0/24

R18#sh run | s r b
router bgp 2042
 bgp log-neighbor-changes
 network 20.0.0.0 mask 255.255.255.0
 neighbor 52.1.0.1 remote-as 520
 neighbor 52.1.0.1 prefix-list pl_LOCAL_ONLY_OUT out
 neighbor 52.1.0.5 remote-as 520
 neighbor 52.1.0.5 prefix-list pl_LOCAL_ONLY_OUT out
 maximum-paths 2
R18#
```

#### Настроим провайдер Киторн так, чтобы в офис Москва отдавался только маршрут по умолчанию.

До фильтрации на R14 было:

```
R14#sh ip bgp 
BGP table version is 10, local router ID is 14.14.14.14
     Network          Next Hop            Metric LocPrf Weight Path
 *>i  20.0.0.0/24      10.10.11.2               0    200      0 301 520 2042 i
 *                     101.0.0.1                              0 101 520 2042 i
 *>i  30.1.0.0/22      10.10.11.2               0    200      0 301 i
 *                     101.0.0.1                              0 101 301 i
 *>i  52.1.0.0/20      10.10.11.2               0    200      0 301 520 i
 *                     101.0.0.1                              0 101 520 i
 *>i  101.0.0.0/21     10.10.11.2               0    200      0 301 101 i
 *                     101.0.0.1                0             0 101 i
 *>   140.100.0.0/24   0.0.0.0                  0         32768 i
 *>i  140.100.0.0/23   10.10.11.2               0    200      0 i
 *>i  140.100.1.0/24   10.10.11.2               0    200      0 i
R14#
```

Видим присутствие марщрутов через AS 101. От провайдера Киторн отдадим в сторону R14 только дефолт :

```
R22:
ip prefix-list pl_DEFAULT-TO-MOSKOW seq 10 permit 0.0.0.0/0

R22#sh run | s r b
router bgp 101
 bgp log-neighbor-changes
 network 101.0.0.0 mask 255.255.248.0
 neighbor 52.1.0.14 remote-as 520
 neighbor 101.0.0.2 remote-as 1001
 neighbor 101.0.0.2 default-originate                          - отдаем дефолт
 neighbor 101.0.0.2 prefix-list pl_DEFAULT-TO-MOSKOW out       - запрещаем все префиксы кроме дефолта
 neighbor 101.0.0.6 remote-as 301
R22#
```

После отсылки дефолта и фильтрации видим только 0.0.0.0 от AS 101:

```
R14#sh ip bgp 
BGP table version is 11, local router ID is 14.14.14.14
     Network          Next Hop            Metric LocPrf Weight Path
 *>   0.0.0.0          101.0.0.1                              0 101 i
 *>i  20.0.0.0/24      10.10.11.2               0    200      0 301 520 2042 i
 *>i  30.1.0.0/22      10.10.11.2               0    200      0 301 i
 *>i  52.1.0.0/20      10.10.11.2               0    200      0 301 520 i
 *>i  101.0.0.0/21     10.10.11.2               0    200      0 301 101 i
 *>   140.100.0.0/24   0.0.0.0                  0         32768 i
 *>i  140.100.0.0/23   10.10.11.2               0    200      0 i
 *>i  140.100.1.0/24   10.10.11.2               0    200      0 i
R14#
```

#### Настроим провайдер Ламас так, чтобы в офис Москва отдавался только маршрут по умолчанию и префикс офиса С.-Петербург.

```
ip prefix-list pl_DEFAULT-TO-MOSKOW seq 10 permit 0.0.0.0/0
ip prefix-list pl_DEFAULT-TO-MOSKOW seq 20 permit 20.0.0.0/24

router bgp 301
 bgp log-neighbor-changes
 network 30.1.0.0 mask 255.255.252.0
 neighbor 30.1.0.2 remote-as 1001
 neighbor 30.1.0.2 default-originate                           - отдаем дефолт
 neighbor 30.1.0.2 prefix-list pl_DEFAULT-TO-MOSKOW out        - заппрещаем все префиксы кроме префиксов СПБ
 neighbor 52.1.0.9 remote-as 520
 neighbor 101.0.0.5 remote-as 101
R21#

