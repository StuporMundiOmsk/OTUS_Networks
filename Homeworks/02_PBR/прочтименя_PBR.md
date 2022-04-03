# PBR и основы маршрутизации с провайдером

## Задание:
1. Пояснить за изменения лабораторки по адресации
2. Настроить взаимодействие Лабытнанги с холодным-холодным окружающим миром
3. Настроить теплый ламповый Чокурдах
4. Настроить видимость Чокурдаха со стороны провайдеров
5. Настроить взаимодействие Чокурдаха с холодным-холодным окружающим миром при условии наличия двух зол
6. Настроить IP SLA в Чокурдахе



## Схема:
![alt-текст](https://github.com/StuporMundiOmsk/OTUS_Networks/blob/main/Homeworks/00_VLAN_Basis/Topology.jpg "Итоговая топология в EVE-NG")


## Решение:

1) На примере внутренней сети Чокурдаха будет видно, что было принято волевое решение использовать маршрутизацию по максимуму. Это будет постепенно находить отражение в изменениях таблицы для лабораторной по адресации. Т.е. никаких лоллипопов и сабынтерфейсов.
Само содержание лабораторной по адресации удобнее оставить именно в виде этакой динамически изменяющейся таблицы, постепенно заполняющейся правильными адресами.
Сети v6 внешнего назначения, для сохранения красоты, съезжают на 80 префикс. Да, не рассчитал. 
Под менеджмент на всех маршрутизаторах будут созданы лупбеки, на коммутаторах - соответствующие виланы.
Линк-локалы настроить не представляется возможным, а потому они отпадают.  


2) Лабытнанги лукаво не мудрствуют, не склоняются и по-простому соединяются с Триадой одним прямым линком. Для того, чтобы жители Лабытнанги смогли посмотреть новую серию любимого сериала, на маршрутизаторах по обе стороны линка должны быть выполнены некоторые нехитрые настройки.    
Со стороны провайдера (согласно принятой политике адресации):
```
!
no ip domain lookup
ip cef
ipv6 unicast-routing
ipv6 cef
!
!
interface Ethernet0/1
 ip address 100.62.1.25 255.255.255.0
 ipv6 address 2001:FFFF:100:62:1::25/80
!
```

Со стороны гордых Лабытнанги (согласно принятой политике адресации):

```
!
no ip domain lookup
ip cef
ipv6 unicast-routing
ipv6 cef
!

!
interface Ethernet0/0
 ip address 100.62.1.27 255.255.255.0
 ipv6 address 2001:FFFF:100:62:1::27/80
!

ip route 0.0.0.0 0.0.0.0 100.62.1.25
!
ipv6 route ::/0 2001:FFFF:100:62:1::25
!
```
Триада увидела Лабытнанги (и умерла. шутка), а из Лабытнанги заметна Триада:
 

3) На SW28 добавлены виланы с адресами, настроены режимы соответствующих портов, маршруты по умолчанию на R28. Хостам заданы адреса и шлюзы.  
```
!
no ip domain-lookup
ip cef
ipv6 unicast-routing
ipv6 cef
!
!
interface Ethernet0/0
 switchport access vlan 101
 switchport mode access
!
interface Ethernet0/1
 switchport access vlan 102
 switchport mode access
!
interface Ethernet0/2
 no switchport
 ip address 10.67.1.29 255.255.255.0
 duplex auto
 ipv6 address 2001:56:67:1::29/64
!
interface Ethernet0/3
!
interface Vlan100
 ip address 10.67.100.29 255.255.255.0
 ipv6 address 2001:56:67:100::29/64
!
interface Vlan101
 ip address 10.67.101.29 255.255.255.0
!
interface Vlan102
 ip address 10.67.102.29 255.255.255.0
!
ip forward-protocol nd
!
no ip http server
no ip http secure-server
!
ip route 0.0.0.0 0.0.0.0 10.67.1.28
!
!
ipv6 route ::/0 2001:56:67:1::28


```
На R28 выполнена комплексная настройка интерфейсов и маршрутов, для обеспечения Чокурдаху внутренней гармонии
```
!
no ip domain lookup
ip cef
ipv6 unicast-routing
ipv6 cef
!
!
!
interface Loopback0
 ip address 10.67.100.28 255.255.255.0
 ipv6 address 2001:56:67:100::28/64
!
interface Ethernet0/0
 ip address 100.67.2.28 255.255.255.0
 ipv6 address 2001:FFFF:100:67:2::28/80
!
interface Ethernet0/1
 ip address 100.67.1.28 255.255.255.0
 shutdown
 ipv6 address 2001:FFFF:100:67:1::28/80
!
interface Ethernet0/2
 ip address 10.67.1.28 255.255.255.0
 ipv6 address 2001:56:67:1::28/64
!
!
ip route 10.67.0.0 255.255.0.0 10.67.1.29
!
ipv6 route 2001:56:67::/48 2001:56:67:1::29
!

```  

Хосты в Чокурдахе ожили и чувствуют себя как дома:
![alt-текст](https://github.com/StuporMundiOmsk/OTUS_Networks/blob/main/Homeworks/02_PBR/%D0%A5%D0%BE%D1%81%D1%82%D1%8B%20%D0%B2%20%D0%A7%D0%BE%D0%BA%D1%80%D1%83%D0%B4%D0%B0%D1%85%D0%B5%20%D0%BE%D0%B6%D0%B8%D0%BB%D0%B8.jpg "Хосты в Чокурдахе ожили")  
   
  
4) На маршрутизаторах со стороны Триады настроены интерфейсы и маршруты для Чокурдаха.   
 
R26:
```  
!
no ip domain lookup
ip cef
ipv6 unicast-routing
ipv6 cef
!
!
interface Ethernet0/1
 ip address 100.67.2.26 255.255.255.0
 ipv6 address 2001:FFFF:100:67:2::26/80
!
!
ip route 10.67.0.0 255.255.0.0 100.67.2.28
!
ipv6 route 2001:56:67::/48 2001:FFFF:100:67:2::28

```  
R25:
```  
!
no ip domain lookup
ip cef
ipv6 unicast-routing
ipv6 cef
!
!
interface Ethernet0/3
 ip address 100.67.1.25 255.255.255.0
 ipv6 address 2001:FFFF:100:67:1::25/80
!
!
ip route 10.67.0.0 255.255.0.0 100.67.1.28
!
ipv6 route 2001:56:67::/48 2001:FFFF:100:67:1::28

```  
5) На R28 настраивается PBR, заворачивающая трафик из 101 сети в 1 канал с Триадой, а трафик из 102 сети - во второй.
``` 
!
ip access-list extended V101
 permit ip 10.67.101.0 0.0.0.255 any
 deny   ip any any
ip access-list extended V102
 permit ip 10.67.102.0 0.0.0.255 any
 deny   ip any any
!
!
route-map ISP permit 10
 match ip address V101
 set ip next-hop 100.67.1.25
!
route-map ISP permit 20
 match ip address V102
 set ip next-hop 100.67.2.26
!  
```
Счетчики завертелись:

![alt-текст](https://github.com/StuporMundiOmsk/OTUS_Networks/blob/main/Homeworks/02_PBR/PBR.jpg "Счетчики завертелись")  

6) На R28 созданы тесты в сторону провайдера.
```
!
ip sla 1
 icmp-jitter 100.67.1.25 source-ip 100.67.1.28 num-packets 5
ip sla schedule 1 life forever start-time now
ip sla 2
 icmp-jitter 100.67.2.26 source-ip 100.67.2.28 num-packets 5
ip sla schedule 2 life forever start-time now
! 
``` 
Тесты отрабатывают удачно:

![alt-текст](https://github.com/StuporMundiOmsk/OTUS_Networks/blob/main/Homeworks/02_PBR/%D0%A2%D0%B5%D1%81%D1%82%D1%8B.jpg "Тесты работают")  

На R28 созданы необходимые треки и внесены изменения в PBR.
```
!
track 1 ip sla 1 reachability
!
track 2 ip sla 2 reachability
!
!
route-map ISP permit 10
 match ip address V101
 set ip next-hop verify-availability 100.67.1.25 1 track 1
 set ip next-hop verify-availability 100.67.2.26 2 track 2
!
route-map ISP permit 20
 match ip address V102
 set ip next-hop verify-availability 100.67.2.26 1 track 2
 set ip next-hop verify-availability 100.67.1.25 2 track 1
!
```
Отключается канал 1. Попытки попасть с хоста PC30 на 100.67.1.25 заворачиваются в 100.67.2.26:
![alt-текст](https://github.com/StuporMundiOmsk/OTUS_Networks/blob/main/Homeworks/02_PBR/%D0%9E%D1%82%D0%BA%D0%BB%D1%8E%D1%87%D0%B5%D0%BD%201.jpg "Отключен 1")  

Отключается канал 2. Попытки попасть из с хоста PC31 на 100.67.2.26 заворачиваются в 100.67.1.25:
![alt-текст](https://github.com/StuporMundiOmsk/OTUS_Networks/blob/main/Homeworks/02_PBR/%D0%9E%D1%82%D0%BA%D0%BB%D1%8E%D1%87%D0%B5%D0%BD%202.jpg "Отключен 2")  
