# IS-IS в K-City 

## Задание:
1. Внедриться в Триаду
2. Развернуть там IS-IS сообразно с условиями


## Схема:
![alt-текст](https://github.com/StuporMundiOmsk/OTUS_Networks/blob/main/Homeworks/03_OSPF/%D0%9C%D0%BE%D1%81%D0%BA%D0%B0%D1%83.jpg "K-City")


## Решение:

1+2) Сперва настраиваются те интерфейсы, которые пока не настроены. Затем, собственно, неведомый и страшный IS-IS разворачивается в недрах провайдера методом последовательной настройки маршрутизаторов

### R23:
```
!
interface Ethernet0/0
 ip address 100.101.1.23 255.255.255.0
 ipv6 address 2001:FFFF:100:101:1::23/80
!
interface Ethernet0/1
 ip address 10.100.1.23 255.255.255.0
 ip router isis
 ipv6 address 2001:56:100:1::23/64
 ipv6 router isis
 isis circuit-type level-2-only
!
interface Ethernet0/2
 ip address 10.100.2.23 255.255.255.0
 ip router isis
 ipv6 address 2001:56:100:2::23/64
 ipv6 router isis
 isis circuit-type level-2-only
!
!
router isis
 net 49.2222.0101.0010.0023.00
 is-type level-2-only
!


```

### R24:
```
!
interface Ethernet0/0
 ip address 100.102.1.24 255.255.255.0
 ipv6 address 2001:FFFF:100:102:1::24/80
!
interface Ethernet0/1
 ip address 10.100.3.24 255.255.255.0
 ip router isis
 ipv6 address 2001:56:100:3::24/64
 ipv6 router isis
 isis circuit-type level-2-only
!
interface Ethernet0/2
 ip address 10.100.2.24 255.255.255.0
 ip router isis
 ipv6 address 2001:56:100:2::24/64
 ipv6 router isis
 isis circuit-type level-2-only
!
interface Ethernet0/3
 ip address 100.40.1.24 255.255.255.0
 ipv6 address 2001:FFFF:100:40:1::24/80
!
!
router isis
 net 49.0024.0101.0010.0024.00
 is-type level-2-only
!


```  

### R25:
``` 
!
interface Ethernet0/0
 ip address 10.100.1.25 255.255.255.0
 ip router isis
 ipv6 address 2001:56:100:1::25/64
 ipv6 router isis
 isis circuit-type level-2-only
!
interface Ethernet0/1
 ip address 100.62.1.25 255.255.255.0
 ipv6 address 2001:FFFF:100:62:1::25/80
!
interface Ethernet0/2
 ip address 10.100.4.25 255.255.255.0
 ip router isis
 ipv6 address 2001:56:100:4::25/64
 ipv6 router isis
 isis circuit-type level-2-only
!
interface Ethernet0/3
 ip address 100.67.1.25 255.255.255.0
 ipv6 address 2001:FFFF:100:67:1::25/80
!
!
router isis
 net 49.2222.0101.0010.0025.00
 is-type level-2-only
 redistribute static ip
!
ip forward-protocol nd
!
!
no ip http server
no ip http secure-server
ip route 10.67.0.0 255.255.0.0 100.67.1.28
!
ipv6 route 2001:56:67::/48 2001:FFFF:100:67:1::28
!


```  
### R26:
``` 
!
interface Ethernet0/0
 ip address 10.100.3.26 255.255.255.0
 ip router isis
 ipv6 address 2001:56:100:3::26/64
 ipv6 router isis
 isis circuit-type level-2-only
!
interface Ethernet0/1
 ip address 100.67.2.26 255.255.255.0
 ipv6 address 2001:FFFF:100:67:2::26/80
!
interface Ethernet0/2
 ip address 10.100.4.26 255.255.255.0
 ip router isis
 ipv6 address 2001:56:100:4::26/64
 ipv6 router isis
 isis circuit-type level-2-only
!
interface Ethernet0/3
 ip address 100.40.2.26 255.255.255.0
 ipv6 address 2001:FFFF:100:40:2::26/80
!
!
router isis
 net 49.0026.0101.0010.0026.00
 is-type level-2-only
 redistribute static ip
!
ip forward-protocol nd
!
!
no ip http server
no ip http secure-server
ip route 10.67.0.0 255.255.0.0 100.67.2.28
!
ipv6 route 2001:56:67::/48 2001:FFFF:100:67:2::28
!


``` 
###Итоговые картинки: 

![alt-текст](https://github.com/StuporMundiOmsk/OTUS_Networks/blob/main/Homeworks/03_OSPF/v4%20%D1%81%20E2.jpg "Neighbors")
![alt-текст](https://github.com/StuporMundiOmsk/OTUS_Networks/blob/main/Homeworks/03_OSPF/v6%20%D0%B1%D0%B5%D0%B7%20E2.jpg "Topologys")
![alt-текст](https://github.com/StuporMundiOmsk/OTUS_Networks/blob/main/Homeworks/03_OSPF/10%20%D0%B7%D0%BE%D0%BD%D0%B0.jpg "Routes")




