# OSPF и доведение до ума внутренней схемы Москау

## Задание:
1. Разобраться, что же там происходит в глубинах Москвы (разумеется, все изменения в адресации отражаются в табличке от первой лабы)
2. Развернуть OSPFы сообразно с условиями: магистральная зона и area 10
3. Развернуть OSPFы сообразно с условиями: зона 101
4. Развернуть OSPFы сообразно с условиями: зона 102




## Схема:
![alt-текст](https://github.com/StuporMundiOmsk/OTUS_Networks/blob/main/Homeworks/03_OSPF/%D0%9C%D0%BE%D1%81%D0%BA%D0%B0%D1%83.jpg "Кусок для Москау")


## Решение:

1) Уже на самых "заниженных" коммутаторах заканчивается L2. Т.е. L2 остается только на доступе, дальше - дивный мир маршрутизации. 
Примечание: я бы сделал по другому и поднял бы L2 выше, если бы не когнитивный диссонанс от простой мысли, что адреса самих виланов тоже должны где-то жить и являться шлюзами, а придумать, где их поселить на классной, во все стороны отказоустойчивой схеме, не получается. К тому же, я задумывал объединить все устройства одной сетью менеджмента, и эта вроде бы простая мысль тоже разбивается вдребезги о схему. Поэтому будем маршрутизировать все и по множеству раз, как завещал Собянин.
###UPD: Свитчи доступа, по рекомендации Алексея, уже будут частью OSPF зоны 10. Рисовать, даже с помощью высоких технологий, - не предназначение грубого неандертальца, коим считаю себя. А потому лучше опишу зоны здесь языком Пушкина и Достоевского.
###Zone 0: L0, E0/0, E0/1, E1/0 маршрутизаторов R14 и R15; E0/2, E0/3 маршрутизаторов R12 и R13.
###Zone 10: L0, E0/0, E0/1 маршрутизаторов R12 и R13; L0, E0/0, E0/1, E1/0, E1/1 маршрутизаторов R4 и R5; L0, E0/0, E0/1 коммутаторов SW2 и SW3.
###Zone 101: E0/3 маршрутизатора R14; L0, E0/0 маршрутизатора R19.
###Zone 102: E0/3 маршрутизатора R15; L0, E0/0 маршрутизатора R20.
1) Интерфейсы  
### Итак, конфиги коммутаторов доступа (L3).
### SW3:
```
!
interface Loopback0
 ip address 10.45.100.3 255.255.255.255
 ip ospf 1 area 10
 ipv6 address 2001:56:45:100::3/128
 ipv6 ospf 1 area 10
!
interface Ethernet0/0
 ip address 10.45.111.3 255.255.255.0
 ip ospf 1 area 10
 ipv6 address 2001:56:45:111::3/64
 ipv6 ospf 1 area 10
!
interface Ethernet0/1
 ip address 10.45.113.3 255.255.255.0
 ip ospf 1 area 10
 ipv6 address 2001:56:45:113::3/64
 ipv6 ospf 1 area 10
!
interface Ethernet0/2
 ip address 10.45.115.3 255.255.255.0
 ip ospf 1 area 10
 ipv6 address 2001:56:45:115::3/64
 ipv6 ospf 1 area 10
!
!
!
router ospf 1
 router-id 3.3.3.3
 passive-interface Ethernet0/2
!
ip forward-protocol nd
!
!
no ip http server
no ip http secure-server
!
ipv6 router ospf 1
 passive-interface Ethernet0/2
!

```
### SW2:
```
!
interface Loopback0
 ip address 10.45.100.2 255.255.255.255
 ipv6 address 2001:56:45:100::2/128
!
interface Ethernet0/0
 ip address 10.45.114.2 255.255.255.0
 ip ospf 1 area 10
 ipv6 address 2001:56:45:114::2/64
 ipv6 ospf 1 area 10
!
interface Ethernet0/1
 ip address 10.45.112.2 255.255.255.0
 ip ospf 1 area 10
 ipv6 address 2001:56:45:112::2/64
 ipv6 ospf 1 area 10
!
interface Ethernet0/2
 ip address 10.45.116.2 255.255.255.0
 ip ospf 1 area 10
 ipv6 address 2001:56:45:116::2/64
 ipv6 ospf 1 area 10
!
!
!
router ospf 1
 router-id 2.2.2.2
 passive-interface Ethernet0/2
!
ip forward-protocol nd
!
!
no ip http server
no ip http secure-server
!
ipv6 router ospf 1
 passive-interface Ethernet0/2
!

```  

### Теперь конфиги маршрутизаторов первой линии
### R4:
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
 ip ospf 1 area 10
 duplex auto
 ipv6 address 2001:56:45:111::4/64
 ipv6 ospf 1 area 10
!
interface Ethernet0/1
 no switchport
 ip address 10.45.112.4 255.255.255.0
 ip ospf 1 area 10
 duplex auto
 ipv6 address 2001:56:45:112::4/64
 ipv6 ospf 1 area 10
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
!
router ospf 1
 router-id 4.4.4.4
!
ip forward-protocol nd
!
no ip http server
no ip http secure-server
!
!
!
ipv6 router ospf 1
!

```  
### R5:
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
 ip ospf 1 area 10
 duplex auto
 ipv6 address 2001:56:45:114::5/64
 ipv6 ospf 1 area 10
!
interface Ethernet0/1
 no switchport
 ip address 10.45.113.5 255.255.255.0
 ip ospf 1 area 10
 duplex auto
 ipv6 address 2001:56:45:113::5/64
 ipv6 ospf 1 area 10
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
!
router ospf 1
 router-id 5.5.5.5
!
ip forward-protocol nd
!
no ip http server
no ip http secure-server
!
!
!
ipv6 router ospf 1
!

``` 

### Теперь конфиги основных маршрутизаторов area 10
### R12:
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
### R13:
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
### Примечание по всем маршрутизаторам и коммутаторам area 10: здесь уже нет никаких статиков - вся надежда на Дейкстру. Видно, что R12 и R13 превратились в ABRы (я так вижу). Итого: в пределах area 10 хосты увидели самые дальние маршрутизаторы, и наоборот. Москва похорошела. 
### Примечание к примечанию: конечно, заколосилось не все. Есть затык неясной природы в куске, где переплелись транспортные сети между коммутаторами и маршрутизаторами нижними. Сами транспортные сети 111-114 не анонсируются, и, почему-то, на маршрутизаторы R12-R13 не прилетают v6 адреса управления коммутаторов. А v4 - прилетают. Сильно подозреваю, что нужно как-то настраивать отдельные экзмепляры OSPFv3. В целом валю все на неудобную схему и непроработку концепции. Считаю, что действительно имеет смысл более конкретно определять рамки: сложность схемы вкупе с дуалстеком и свободной фантазией студента приводят к слишком большой боли... 


![alt-текст](https://github.com/StuporMundiOmsk/OTUS_Networks/blob/main/Homeworks/03_OSPF/v4%20%D1%81%20E2.jpg "v4 маршруты на R12 и R13")
![alt-текст](https://github.com/StuporMundiOmsk/OTUS_Networks/blob/main/Homeworks/03_OSPF/v4%20%D1%81%20E2.jpg "v6 маршруты на R12 и R13")
![alt-текст](https://github.com/StuporMundiOmsk/OTUS_Networks/blob/main/Homeworks/03_OSPF/10%20%D0%B7%D0%BE%D0%BD%D0%B0.jpg "Пинги от хостов до краев 10 зоны")



2) Настройка основных маршрутизаторов магистральной зоны и распространение маршрутов по умолчанию (зоны 101 и 102 пока не созданы)
### Конфиг R14:
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

### Конфиг R15:
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

### Примечание к основным маршрутизаторам магистральной зоны: т.к. и R14 и R15 ещё и ASBRы по схеме, то маршруты по умолчанию будут на обоих. Соответственно, чтобы по-человечески распространять умолчания по Москве, принято волевое решение считать маршруты R15 за маршруты второго сорта, присвоив им AD больше, чем принято для OSPF.  
![alt-текст](https://github.com/StuporMundiOmsk/OTUS_Networks/blob/main/Homeworks/03_OSPF/0%20%D0%B7%D0%BE%D0%BD%D0%B0%20%D0%BE%D1%82%20%D1%85%D0%BE%D1%81%D1%82%D0%BE%D0%B2.jpg "Пинги от хостов до краев 0 зоны")

![alt-текст](https://github.com/StuporMundiOmsk/OTUS_Networks/blob/main/Homeworks/03_OSPF/R14.jpg "v4 маршруты на R14 и R15")
![alt-текст](https://github.com/StuporMundiOmsk/OTUS_Networks/blob/main/Homeworks/03_OSPF/R15.jpg "v6 vаршруты на R14 и R15")



3) Зона 101
### Конфиг R19:
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
 area 101 stub
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

### Добавления в конфиг R14:
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
 area 101 stub
 area 101 filter-list prefix DEFAULTS_ONLY in
 default-information originate
!
!
!
ip prefix-list DEFAULTS_ONLY seq 5 permit 0.0.0.0/0

``` 
### Примечания к зоне 101: да, такой нехитрый фильтр должен блокировать все, кроме префикса с нулями. 
![alt-текст](https://github.com/StuporMundiOmsk/OTUS_Networks/blob/main/Homeworks/03_OSPF/R19.jpg "Маршруты на R19")




4) Зона 102
### Конфиг на R20:
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
### Добавления в конфиг R15:
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
![alt-текст](https://github.com/StuporMundiOmsk/OTUS_Networks/blob/main/Homeworks/03_OSPF/R20.jpg "Маршруты на R20")
