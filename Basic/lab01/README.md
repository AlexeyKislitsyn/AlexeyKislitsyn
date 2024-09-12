# Лабораторная работа. Базовая настройка коммутатора

### Топология
![](1.png)

### Таблица адресации
```
    S1  -  VLAN1  -  192.168.1.2/24
   PC-A -   NIC   -  192.168..10/24  
```
# Часть 1. Создание сети и проверка настроек коммутатора по умолчанию
### Шаг 1  Создайте сеть согласно топологии
  Выполнено создание сети согласно топпологии. Установлено консольное подключение к коммутатору с помощью программы эмуляции терминала.
  Консольное подключение используется при начальной настройке коммутатора не имеющего сетевого доступа.Только после начальной настройки коммутатора мы можем использовать удаленное подключение Telnet, SSH. 
### Шаг 2  Проверьте настройки коммутатора по умолчанию. 
  a. Просмотр show runnin-cinfig. Для полноты сбросим к заводским настройкам, если есть vlan.dat удалим его и перезагрузим коммутатор. Используем команды:
 ```  
   Switch# show flash
   Switch# delete vlan.dat
   Switch#erase startup-config. 
   Switch#reload
 ```
  b. Изучите текущий файл running configuration
   Имеем: 24 интерфейса FastEthernet, 2 интерфейса GigabitEthernet
   диапазон значений, отображаемых в vty-линиях: 0-4, 5-15
   
  с. Изучите файл загрузочной конфигурации (startup configuration) 
  ``` 
   Switch#sh startup-config 
   startup-config is not present
  ``` 
   Данное сообщение говорит о том, что еще не производилась запись конфига в  startup-config 
   
   d. Изучите характеристики SVI для VLAN 1.
    
    Switch#sh int vlan 1
    
    ip для VLAN 1 еще не назначен
    mac vlan 1 :  0010.11e9.703b
    интерфейс отключен:  Vlan1 is administratively down, line protocol is down

   e. Изучите IP-свойства интерфейса SVI сети VLAN 1
   
    Switch#sh ip int vlan 1
   
   g. Изучите сведения о версии ОС Cisco IOS на коммутаторе.
    
    Switch#sh ver
    Switch#sh flash:
   
    Cisco IOS  :  C2960 Software (C2960-LANBASEK9-M), Version 15.0(2)SE4
    файл образа :  System image file is "flash:c2960-lanbasek9-mz.150-2.SE4.bin"

   h. Изучите свойства по умолчанию интерфейса FastEthernet, который используется компьютером PC-A.
   
    Switch#sh int fast 0/6
   
     по умолчанию интерфейс в down : FastEthernet0/6 is down, line protocol is down (disabled)
     включить интерфейс: Switch(config-if)# no shutdown 
     mac 0001.4333.4506
     Full-duplex, 100Mb/s

    i. Изучите флеш-память.
    
    Switch#flash:
    Switch#sh dir
    
     файл образа :  System image file is "flash:c2960-lanbasek9-mz.150-2.SE4.bin"


   
