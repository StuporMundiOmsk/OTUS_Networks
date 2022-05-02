# OSPF и доведение до ума внутренней схемы Москау

## Задание:
1. Разобраться, что же там происходит в глубинах Москвы (разумеется, все изменения в адресации отражаются в табличке от первой лабы)
2. Равернуть OSPFы сообразно с условиями: магистральная зона и area 10
3. Развернуть OSPFы сообразно с условиями: зона 101
4. Развернуть OSPFы сообразно с условиями: зона 102




## Схема:
![alt-текст](https://github.com/StuporMundiOmsk/OTUS_Networks/blob/main/Homeworks/02_PBR/Topolgy_PBR.jpg "Кусок для Москау")


## Решение:

1) Уже на самых "заниженных" коммутаторах заканчивается L2. Т.е. L2 остается только на доступе, дальше - дивный мир маршрутизации. 
Примечание: я бы сделал по другому и поднял бы L2 выше, если бы не когнитивный диссонанс от простой мысли, что адреса самих виланов тоже должны где-то жить и являться шлюзами, а придумать, где их поселить на классной, во все стороны отказоустойчивой схеме, не получается. К тому же, я задумывал объеденить все устройства одной сетью менеджмента, и эта вроде бы простая мысль тоже разбивается вдребезги о схему. Поэтому будем маршрутизировать все и по множеству раз, как завещал Собянин.
###Итак, конфиги коммутаторов доступа (L2+).
###SW3:
```
!
interface Loopback0
 ip address 10.45.100.3 255.255.255.255
 ipv6 address 2001:56:45:100::3/128
!
interface Ethernet0/0
 no switchport
 ip address 10.45.111.3 255.255.255.0
 duplex auto
 ipv6 address 2001:56:45:111::3/64
!
interface Ethernet0/1
 no switchport
 ip address 10.45.113.3 255.255.255.0
 duplex auto
 ipv6 address 2001:56:45:113::3/64
!
interface Ethernet0/2
 switchport access vlan 115
 switchport mode access
!
!
interface Vlan115
 ip address 10.45.115.3 255.255.255.0
 ipv6 address 2001:56:45:115::3/64
!
ip forward-protocol nd
!
no ip http server
no ip http secure-server
!
ip route 0.0.0.0 0.0.0.0 10.45.111.4
ip route 0.0.0.0 0.0.0.0 10.45.113.5 10
!
!
ipv6 route ::/0 2001:56:45:111::4
ipv6 route ::/0 2001:56:45:113::5 10
```
###SW2:
```
!
interface Loopback0
 ip address 10.45.100.2 255.255.255.255
 ipv6 address 2001:56:45:100::2/128
!
interface Ethernet0/0
 no switchport
 ip address 10.45.114.2 255.255.255.0
 duplex auto
 ipv6 address 2001:56:45:114::2/64
!
interface Ethernet0/1
 no switchport
 ip address 10.45.112.2 255.255.255.0
 duplex auto
 ipv6 address 2001:56:45:112::2/64
!
interface Ethernet0/2
 switchport access vlan 116
 switchport mode access
!
!
interface Vlan116
 ip address 10.45.116.2 255.255.255.0
 ipv6 address 2001:56:45:116::2/64
!
ip forward-protocol nd
!
no ip http server
no ip http secure-server
!
ip route 0.0.0.0 0.0.0.0 10.45.114.5
ip route 0.0.0.0 0.0.0.0 10.45.112.4 10
!
!
ipv6 route ::/0 2001:56:45:112::4 10
ipv6 route ::/0 2001:56:45:114::5
```  
###Примечание по коммутаторам: маршруты выше с уровня доступа организованы простой избыточностью умолчаний с разными метриками. Не самое годное решение, но городить PBR, который не будет работать с v6, тоже странно.

###Теперь конфиги маршрутизаторов первой линии
###R4:
``` 
!
interface Loopback0
 ip address 10.45.100.4 255.255.255.255
 ip ospf 1 area 10
 ipv6 address 2001:56:45:100::4/128
 ipv6 ospf 1 area 10
!
interface Ethernet0/0
 no switchport
 ip address 10.45.111.4 255.255.255.0
 duplex auto
 ipv6 address 2001:56:45:111::4/64
!
interface Ethernet0/1
 no switchport
 ip address 10.45.112.4 255.255.255.0
 duplex auto
 ipv6 address 2001:56:45:112::4/64
!
interface Ethernet0/2
!
interface Ethernet0/3
!
interface Ethernet1/0
 no switchport
 ip address 10.45.7.4 255.255.255.0
 ip ospf 1 area 10
 duplex auto
 ipv6 address 2001:56:45:7::4/64
 ipv6 ospf 1 area 10
!
interface Ethernet1/1
 no switchport
 ip address 10.45.9.4 255.255.255.0
 ip ospf 1 area 10
 duplex auto
 ipv6 address 2001:56:45:9::4/64
 ipv6 ospf 1 area 10
!
!
router ospf 1
 router-id 4.4.4.4
 redistribute static subnets
!
ip forward-protocol nd
!
no ip http server
no ip http secure-server
!
ip route 10.45.100.2 255.255.255.255 10.45.112.2
ip route 10.45.100.3 255.255.255.255 10.45.111.3
ip route 10.45.115.0 255.255.255.0 10.45.111.3
ip route 10.45.116.0 255.255.255.0 10.45.112.2
!
!
ipv6 route 2001:56:45:100::2/128 2001:56:45:112::2
ipv6 route 2001:56:45:100::3/128 2001:56:45:111::3
ipv6 router ospf 1
!
!
```  
###R5:
``` 
!
interface Loopback0
 ip address 10.45.100.5 255.255.255.255
 ip ospf 1 area 10
 ipv6 address 2001:56:45:100::5/128
 ipv6 ospf 1 area 10
!
interface Ethernet0/0
 no switchport
 ip address 10.45.114.5 255.255.255.0
 duplex auto
 ipv6 address 2001:56:45:114::5/64
!
interface Ethernet0/1
 no switchport
 ip address 10.45.113.5 255.255.255.0
 duplex auto
 ipv6 address 2001:56:45:113::5/64
!
interface Ethernet0/2
!
interface Ethernet0/3
!
interface Ethernet1/0
 no switchport
 ip address 10.45.10.5 255.255.255.0
 ip ospf 1 area 10
 duplex auto
 ipv6 address 2001:56:45:10::5/64
 ipv6 ospf 1 area 10
!
interface Ethernet1/1
 no switchport
 ip address 10.45.8.5 255.255.255.0
 ip ospf 1 area 10
 duplex auto
 ipv6 address 2001:56:45:8::5/64
 ipv6 ospf 1 area 10
!
!
router ospf 1
 router-id 5.5.5.5
 redistribute static subnets
!
ip forward-protocol nd
!
no ip http server
no ip http secure-server
!
ip route 10.45.100.2 255.255.255.255 10.45.114.2
ip route 10.45.100.3 255.255.255.255 10.45.113.3
ip route 10.45.115.0 255.255.255.0 10.45.113.3
ip route 10.45.116.0 255.255.255.0 10.45.114.2
!
!
ipv6 route 2001:56:45:100::2/128 2001:56:45:114::2
ipv6 route 2001:56:45:100::3/128 2001:56:45:113::3
ipv6 router ospf 1
!
``` 
###Примечание по маршрутизаторам первой линии: да, уже здесь обнаруживается разросшаяся area 10. Статикой задаются маршруты в сети доступа и к адресам управления, затем анонсируются скопом (что не очень хорошо, но заморачиваться с дорожными картами здесь - ломать голову). 

###Теперь конфиги основных маршрутизаторов area 10
###R12:
``` 
!
interface Loopback0
 ip address 10.45.100.12 255.255.255.255
 ip ospf 1 area 10
 ipv6 address 2001:56:45:100::12/128
 ipv6 ospf 1 area 10
!
interface Ethernet0/0
 ip address 10.45.7.12 255.255.255.0
 ip ospf 1 area 10
 ipv6 address 2001:56:45:7::12/64
 ipv6 ospf 1 area 10
!
interface Ethernet0/1
 ip address 10.45.8.12 255.255.255.0
 ip ospf 1 area 10
 ipv6 address 2001:56:45:8::12/64
 ipv6 ospf 1 area 10
!
interface Ethernet0/2
 ip address 10.45.2.12 255.255.255.0
 ip ospf 1 area 0
 ipv6 address 2001:56:45:2::12/64
 ipv6 ospf 1 area 0
!
interface Ethernet0/3
 ip address 10.45.4.12 255.255.255.0
 ip ospf 1 area 0
 ipv6 address 2001:56:45:4::12/64
 ipv6 ospf 1 area 0
!
!
router ospf 1
 router-id 12.12.12.12
!
ip forward-protocol nd
!
!
no ip http server
no ip http secure-server
!
ipv6 router ospf 1

``` 
###R13:
``` 
!
interface Loopback0
 ip address 10.45.100.13 255.255.255.255
 ip ospf 1 area 10
 ipv6 address 2001:56:45:100::13/128
 ipv6 ospf 1 area 10
!
interface Ethernet0/0
 ip address 10.45.10.13 255.255.255.0
 ip ospf 1 area 10
 ipv6 address 2001:56:45:10::13/64
 ipv6 ospf 1 area 10
!
interface Ethernet0/1
 ip address 10.45.9.13 255.255.255.0
 ip ospf 1 area 10
 ipv6 address 2001:56:45:9::13/64
 ipv6 ospf 1 area 10
!
interface Ethernet0/2
 ip address 10.45.5.13 255.255.255.0
 ip ospf 1 area 0
 ipv6 address 2001:56:45:5::13/64
 ipv6 ospf 1 area 0
!
interface Ethernet0/3
 ip address 10.45.3.13 255.255.255.0
 ip ospf 1 area 0
 ipv6 address 2001:56:45:3::13/64
 ipv6 ospf 1 area 0
!
!
router ospf 1
 router-id 13.13.13.13
!
ip forward-protocol nd
!
!
no ip http server
no ip http secure-server
!
ipv6 router ospf 1
!

```  
###Примечание по основным маршрутизаторам area 10: здесь уже нет никаких статиков - вся надежда на Дейкстру. Видно, что оба они превратились в ABRы (я так вижу). Итого: в пределах area 10 хосты увидели самые дальние маршрутизаторы, и наоборот. Москва похорошела. 
###Примечание к примечанию: конечно, заколосилось не все. Есть затык неясной природы в куске, где переплелись транспортные сети между коммутаторами и маршрутизаторами нижними. Сами транспортные сети 111-114 не анонсируются, и, почему-то, на маршрутизаторы R12-R13 не прилетают v6 адреса управления коммутаторов. А v4 - прилетают. Сильно подозреваю, что нужно как-то настраивать отдельные экзмепляры OSPFv3. В целом валю все на неудобную схему и непроработку концепции. Считаю, что действительно имеет смысл более конкретно определять рамки: сложность схемы вкупе с дуалстеком и свободной фантазией студента приводят к слишком большой боли... 


![alt-текст](https://github.com/StuporMundiOmsk/OTUS_Networks/blob/main/Homeworks/02_PBR/Topolgy_PBR.jpg "Прилетает .100.2 и .100.3,...")
![alt-текст](https://github.com/StuporMundiOmsk/OTUS_Networks/blob/main/Homeworks/02_PBR/Topolgy_PBR.jpg "...а :100::2 и :100::3 не прилетает")
![alt-текст](https://github.com/StuporMundiOmsk/OTUS_Networks/blob/main/Homeworks/02_PBR/Topolgy_PBR.jpg "Пинги от хостов до краев 10 зоны")



2) Настройка основных маршрутизаторов магистральной зоны и распространение маршрутов по умолчанию (зоны 101 и 102 пока не созданы)
###Конфиг R14:
```
!
!
interface Loopback0
 ip address 10.45.100.14 255.255.255.255
 ip ospf 1 area 0
 ipv6 address 2001:56:45:100::14/128
 ipv6 ospf 1 area 0
!
interface Ethernet0/0
 ip address 10.45.2.14 255.255.255.0
 ip ospf 1 area 0
 ipv6 address 2001:56:45:2::14/64
 ipv6 ospf 1 area 0
!
interface Ethernet0/1
 ip address 10.45.3.14 255.255.255.0
 ip ospf 1 area 0
 ipv6 address 2001:56:45:3::14/64
 ipv6 ospf 1 area 0
!
interface Ethernet0/2
 ip address 101.45.1.14 255.255.255.0
 ipv6 address 2001:FFFF:101:45:1::14/80
!
interface Ethernet0/3
 ip address 10.45.1.14 255.255.255.0
 ipv6 address 2001:56:45:1::14/64
!
interface Ethernet1/0
 ip address 10.45.20.14 255.255.255.0
 ip ospf 1 area 0
 ipv6 address 2001:56:45:20::14/64
 ipv6 ospf 1 area 0
!
router ospf 1
 router-id 14.14.14.14
 default-information originate
!
ip forward-protocol nd
!
!
no ip http server
no ip http secure-server
ip route 0.0.0.0 0.0.0.0 101.45.1.22
!
ipv6 route ::/0 2001:FFFF:101:45:1::22
ipv6 router ospf 1
!
```

###Конфиг R15:
```
!
interface Loopback0
 ip address 10.45.100.15 255.255.255.255
 ip ospf 1 area 0
 ipv6 address 2001:56:45:100::15/128
 ipv6 ospf 1 area 0
!
interface Ethernet0/0
 ip address 10.45.5.15 255.255.255.0
 ip ospf 1 area 0
 ipv6 address 2001:56:45:5::15/64
 ipv6 ospf 1 area 0
!
interface Ethernet0/1
 ip address 10.45.4.15 255.255.255.0
 ip ospf 1 area 0
 ipv6 address 2001:56:45:4::15/64
 ipv6 ospf 1 area 0
!
interface Ethernet0/2
 ip address 102.45.1.15 255.255.255.0
 ipv6 address 2001:FFFF:102:45:1::15/80
!
interface Ethernet0/3
 ip address 10.45.6.15 255.255.255.0
 ipv6 address 2001:56:45:6::15/64
!
interface Ethernet1/0
 ip address 10.45.20.15 255.255.255.0
 ip ospf 1 area 0
 ipv6 address 2001:56:45:20::15/64
 ipv6 ospf 1 area 0
!
!
router ospf 1
 router-id 15.15.15.15
 default-information originate
!
ip forward-protocol nd
!
!
no ip http server
no ip http secure-server
ip route 0.0.0.0 0.0.0.0 102.45.1.21 120
!
ipv6 route ::/0 2001:FFFF:102:45:1::21 120
ipv6 router ospf 1
!
```

###Примечание к основным маршрутизаторам магистральной зоны: т.к. и R14 и R15 ещё и ASBRы по схеме, то маршруты по умолчанию будут на обоих. Соответственно, чтобы по-человечески распространять умолчания по Москве, принято волевое решение считать маршруты R15 за маршруты второго сорта, присвоив им AD больше, чем принято для OSPF. 
Примечание к примечанию: да, v6 снова не пожелали распространяться. Подозрения про отдельные экземпляры OSPFv3 усилились...  
![alt-текст](https://github.com/StuporMundiOmsk/OTUS_Networks/blob/main/Homeworks/02_PBR/Topolgy_PBR.jpg "Пинги от хостов до краев 0 зоны")
![alt-текст](https://github.com/StuporMundiOmsk/OTUS_Networks/blob/main/Homeworks/02_PBR/Topolgy_PBR.jpg "Маршруты на R4")
![alt-текст](https://github.com/StuporMundiOmsk/OTUS_Networks/blob/main/Homeworks/02_PBR/Topolgy_PBR.jpg "Маршруты на R14")
![alt-текст](https://github.com/StuporMundiOmsk/OTUS_Networks/blob/main/Homeworks/02_PBR/Topolgy_PBR.jpg "Маршруты на R15")



3) Зона 101
###Конфиг R19:
```
!
interface Loopback0
 ip address 10.45.100.19 255.255.255.255
 ip ospf 1 area 101
 ipv6 address 2001:56:45:100::19/64
 ipv6 ospf 1 area 101
!
interface Ethernet0/0
 ip address 10.45.1.19 255.255.255.0
 ip ospf 1 area 101
 ipv6 address 2001:56:45:1::19/64
!
router ospf 1
 router-id 19.19.19.19
!
ip forward-protocol nd
!
!
no ip http server
no ip http secure-server
!
ipv6 router ospf 1
!

``` 

###Добавления в конфиг R14:
``` 
!
interface Ethernet0/3
 ip address 10.45.1.14 255.255.255.0
 ip ospf 1 area 101
 ipv6 address 2001:56:45:1::14/64
 ipv6 ospf 1 area 101
!
!
!
router ospf 1
 router-id 14.14.14.14
 area 101 filter-list prefix DEFAULTS_ONLY in
 default-information originate
!
!
!
ip prefix-list DEFAULTS_ONLY seq 5 permit 0.0.0.0/0

``` 
###Примечания к зоне 101: да, такой нехитрый фильтр должен блокировать все, кроме префикса с нулями. Однако, E2 маршруты ему неподвластны. При этом зарезать конкретно эти E2 где-то раньше не представляется возможным, т.к. префиксы эти важные. 
![alt-текст](https://github.com/StuporMundiOmsk/OTUS_Networks/blob/main/Homeworks/02_PBR/Topolgy_PBR.jpg "Маршруты на R19")




4) Зона 102
###Конфиг на R20:
``` 
!
interface Loopback0
 ip address 10.45.100.20 255.255.255.255
 ip ospf 1 area 102
 ipv6 address 2001:56:45:100::20/64
 ipv6 ospf 1 area 102
!
interface Ethernet0/0
 ip address 10.45.6.20 255.255.255.0
 ip ospf 1 area 102
 ipv6 address 2001:56:45:6::20/64
 ipv6 ospf 1 area 102
!
router ospf 1
 router-id 20.20.20.20
!
ip forward-protocol nd
!
!
no ip http server
no ip http secure-server
!
ipv6 router ospf 1
!

``` 
###Добавления в конфиг R15:
``` 
!
interface Ethernet0/3
 ip address 10.45.6.15 255.255.255.0
 ip ospf 1 area 102
 ipv6 address 2001:56:45:6::15/64
 ipv6 ospf 1 area 102
!
!
!
router ospf 1
 router-id 15.15.15.15
 area 102 filter-list prefix DENY_101 in
 default-information originate
!
ip forward-protocol nd
!
!
no ip http server
no ip http secure-server
ip route 0.0.0.0 0.0.0.0 102.45.1.21 120
!
!
ip prefix-list DENY_101 seq 5 deny 10.45.1.0/24
ip prefix-list DENY_101 seq 10 deny 10.45.100.19/32
ip prefix-list DENY_101 seq 15 permit 0.0.0.0/0 le 32
ipv6 route ::/0 2001:FFFF:102:45:1::21 120
ipv6 router ospf 1

``` 
![alt-текст](https://github.com/StuporMundiOmsk/OTUS_Networks/blob/main/Homeworks/02_PBR/Topolgy_PBR.jpg "Маршруты на R20")
