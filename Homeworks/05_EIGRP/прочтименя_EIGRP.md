# EIGRP в Ленинграде 

## Задание:
1. Настроить адреса в Ленинграде
2. Развернуть там EIGRP без ограничений
3. Суммировать распространяемые префиксы на R16 и R17 во имя удобства и красоты
4. Ущемить максимально права отщепенца R32
5. Исправление нестыковок с IPv6


## Схема:
![alt-текст](https://github.com/StuporMundiOmsk/OTUS_Networks/blob/main/Homeworks/05_EIGRP/Leningrad.jpg "Ленинград")


## Решение:

## 1) Адреса на оборудовании, согласно наконец-то окончательно сформировавшейся табличке из лабораторки по адресации:
Базисная табличка: https://github.com/StuporMundiOmsk/OTUS_Networks/blob/main/Homeworks/01_Addresses/%D0%BF%D1%80%D0%BE%D1%87%D1%82%D0%B8%D0%BC%D0%B5%D0%BD%D1%8F.md

### SW9:
```
!
interface Loopback0
 ip address 10.40.100.9 255.255.255.255
 ipv6 address 2001:56:40:100::9/64
!
!
interface Ethernet0/2
 ip address 10.40.108.9 255.255.255.0
 ipv6 address 2001:56:40:108::9/64
!
interface Ethernet0/3
 ip address 10.40.4.9 255.255.255.0
 ipv6 address 2001:56:40:4::9/64
!
interface Ethernet1/0
 ip address 10.40.6.9 255.255.255.0
 ipv6 address 2001:56:40:6::9/64
!
```

### SW10:
```
!
interface Loopback0
 ip address 10.40.100.10 255.255.255.255
 ipv6 address 2001:56:40:100::10/64
!
!
interface Ethernet0/2
 ip address 10.40.109.10 255.255.255.0
 ipv6 address 2001:56:40:109::10/64
!
interface Ethernet0/3
 ip address 10.40.7.10 255.255.255.0
 ipv6 address 2001:56:40:7::10/64
!
interface Ethernet1/0
 ip address 10.40.5.10 255.255.255.0
 ipv6 address 2001:56:40:5::10/64
!
```  

### R16:
``` 
!
interface Loopback0
 ip address 10.40.100.16 255.255.255.255
 ipv6 address 2001:56:40:100::16/64
!
interface Ethernet0/0
 ip address 10.40.7.16 255.255.255.0
 ipv6 address 2001:56:40:7::16/64
!
interface Ethernet0/1
 ip address 10.40.2.16 255.255.255.0
 ipv6 address 2001:56:40:2::16/64
!
interface Ethernet0/2
 ip address 10.40.6.16 255.255.255.0
 ipv6 address 2001:56:40:6::16/64
!
interface Ethernet0/3
 ip address 10.40.3.16 255.255.255.0
 ipv6 address 2001:56:40:3::16/64
!
``` 
 
### R17:
``` 
!
interface Loopback0
 ip address 10.40.100.17 255.255.255.255
 ipv6 address 2001:56:40:100::17/64
!
interface Ethernet0/0
 ip address 10.40.4.17 255.255.255.0
 ipv6 address 2001:56:40:4::17/64
!
interface Ethernet0/1
 ip address 10.40.1.17 255.255.255.0
 ipv6 address 2001:56:40:1::17/64
!
interface Ethernet0/2
 ip address 10.40.5.17 255.255.255.0
 ipv6 address 2001:56:40:5::17/64
```
 
### R32:
``` 
!
interface Loopback0
 ip address 10.40.100.32 255.255.255.255
 ipv6 address 2001:56:40:100::32/64
!
interface Ethernet0/0
 ip address 10.40.3.32 255.255.255.0
 ipv6 address 2001:56:40:3::32/64
!
```

### R18:
``` 
!
interface Loopback0
 ip address 10.40.100.18 255.255.255.0
 ipv6 address 2001:56:40:100::18/64
!
interface Ethernet0/0
 ip address 10.40.2.18 255.255.255.0
 ipv6 address 2001:56:40:2::18/64
!
interface Ethernet0/1
 ip address 10.40.1.18 255.255.255.0
 ipv6 address 2001:56:40:1::18/64
!
interface Ethernet0/2
 ip address 100.40.1.18 255.255.255.0
 ipv6 address 2001:FFFF:100:40:1::18/80
!
interface Ethernet0/3
 ip address 100.40.2.18 255.255.255.0
 ipv6 address 2001:FFFF:100:40:2::18/80
!
ip route 0.0.0.0 0.0.0.0 100.40.1.24
ip route 0.0.0.0 0.0.0.0 100.40.2.26 10
!
ipv6 route ::/0 2001:FFFF:100:40:2::26 10
ipv6 route ::/0 2001:FFFF:100:40:1::24
!

```


## 2) Настройка максимального именованного EIGRP (уже с суммированием и фильтрацией)
### SW9:
```
!
router eigrp Lenin
 !
 address-family ipv4 unicast autonomous-system 1
  !
  af-interface Ethernet0/2
   passive-interface
  exit-af-interface
  !
  topology base
  exit-af-topology
  network 10.40.4.0 0.0.0.255
  network 10.40.6.0 0.0.0.255
  network 10.40.100.9 0.0.0.0
  network 10.40.108.0 0.0.0.255
  eigrp router-id 9.9.9.9
 exit-address-family
 !
 address-family ipv6 unicast autonomous-system 1
  !
  af-interface Ethernet0/2
   passive-interface
  exit-af-interface
  !
  topology base
  exit-af-topology
  eigrp router-id 9.9.9.9
 exit-address-family
!

```

### SW10:
```
!
router eigrp Lenin
 !
 address-family ipv4 unicast autonomous-system 1
  !
  af-interface Ethernet0/2
   passive-interface
  exit-af-interface
  !
  topology base
  exit-af-topology
  network 10.40.5.0 0.0.0.255
  network 10.40.7.0 0.0.0.255
  network 10.40.100.10 0.0.0.0
  network 10.40.109.0 0.0.0.255
  eigrp router-id 10.10.10.10
 exit-address-family
 !
 address-family ipv6 unicast autonomous-system 1
  !
  af-interface Ethernet0/2
   passive-interface
  exit-af-interface
  !
  topology base
  exit-af-topology
 exit-address-family
!

```  

### R16:
``` 
!
router eigrp Lenin
 !
 address-family ipv4 unicast autonomous-system 1
  !
  af-interface Ethernet0/1
   summary-address 10.40.0.0 255.255.0.0
  exit-af-interface
  !
  topology base
   distribute-list 1 out Ethernet0/3
  exit-af-topology
  network 10.40.2.0 0.0.0.255
  network 10.40.3.0 0.0.0.255
  network 10.40.6.0 0.0.0.255
  network 10.40.7.0 0.0.0.255
  network 10.40.100.16 0.0.0.0
  eigrp router-id 16.16.16.16
 exit-address-family
 !
 address-family ipv6 unicast autonomous-system 1
  !
  af-interface Ethernet0/1
   summary-address 2001:56:40::/48
  exit-af-interface
  !
  topology base
  exit-af-topology
  eigrp router-id 16.16.16.16
 exit-address-family
!
!
access-list 1 permit 0.0.0.0
!


```  
### R17:
``` 
!
router eigrp Lenin
 !
 address-family ipv4 unicast autonomous-system 1
  !
  af-interface Ethernet0/1
   summary-address 10.40.0.0 255.255.0.0
  exit-af-interface
  !
  topology base
  exit-af-topology
  network 10.40.1.0 0.0.0.255
  network 10.40.4.0 0.0.0.255
  network 10.40.5.0 0.0.0.255
  network 10.40.100.17 0.0.0.0
  eigrp router-id 17.17.17.17
 exit-address-family
 !
 address-family ipv6 unicast autonomous-system 1
  !
  af-interface Ethernet0/1
   summary-address 2001:56:40::/48
  exit-af-interface
  !
  topology base
  exit-af-topology
  eigrp router-id 17.17.17.17
 exit-address-family
!

``` 

### R32:
``` 
!
router eigrp Lenin
 !
 address-family ipv4 unicast autonomous-system 1
  !
  topology base
  exit-af-topology
  network 10.40.3.0 0.0.0.255
  network 10.40.100.32 0.0.0.0
  eigrp router-id 32.32.32.32
 exit-address-family
 !
 address-family ipv6 unicast autonomous-system 1
  !
  topology base
  exit-af-topology
  eigrp router-id 32.32.32.32
 exit-address-family
!

```

### R18:
``` 
!
router eigrp Lenin
 !
 address-family ipv4 unicast autonomous-system 1
  !
  af-interface Ethernet0/2
   passive-interface
  exit-af-interface
  !
  af-interface Ethernet0/3
   passive-interface
  exit-af-interface
  !
  topology base
   redistribute static
  exit-af-topology
  network 10.40.1.0 0.0.0.255
  network 10.40.2.0 0.0.0.255
  network 10.40.100.18 0.0.0.0
  network 100.40.1.0 0.0.0.255
  network 100.40.2.0 0.0.0.255
  eigrp router-id 18.18.18.18
 exit-address-family
 !
 address-family ipv6 unicast autonomous-system 1
  !
  af-interface Ethernet0/2
   passive-interface
  exit-af-interface
  !
  af-interface Ethernet0/3
   passive-interface
  exit-af-interface
  !
  topology base
   redistribute static
  exit-af-topology
  eigrp router-id 18.18.18.18
 exit-address-family
!

```


## 3) На R16 и R17 вручную суммируются маршруты для передачи на вышестоящий R18, что наглядно подтверждается красивой RIB. R16 и R17 имеют более серьезные по размеру RIBы (включая умолчания от R18), маршрутизация в пределах Ленинграда работает!
![alt-текст](https://github.com/StuporMundiOmsk/OTUS_Networks/blob/main/Homeworks/05_EIGRP/R18_routes.jpg "R18_routes")
![alt-текст](https://github.com/StuporMundiOmsk/OTUS_Networks/blob/main/Homeworks/05_EIGRP/R16_v4_routes.jpg "R16_v4_routes") 
![alt-текст](https://github.com/StuporMundiOmsk/OTUS_Networks/blob/main/Homeworks/05_EIGRP/R16_v6_routes.jpg "R16_v6_routes") 
![alt-текст](https://github.com/StuporMundiOmsk/OTUS_Networks/blob/main/Homeworks/05_EIGRP/Leningrads_routing.jpg "Leningrads_routing")


## 4) Для ущемления горемычного R32 применяется фильтрация на основе ACL (см. конфиг R16). С v4 все работает, как надо, а вот с v6 возникли проблемы: во-первых, статический v6 почему-то не желает редистрибьютиться с R18; а во-вторых, я не очень понимаю, как его фильтровать.
![alt-текст](https://github.com/StuporMundiOmsk/OTUS_Networks/blob/main/Homeworks/05_EIGRP/R32_v4_routes.jpg "R32_v4_routes") 

  
## 5) По совету Алексея в умолчаниях для v6 на R18 глобальные адреса были заменены на линк-локалы (да, т.к. они не уникальны, пришлось указывать интерфейсы), и сразу же R32 увидел свет в конце v6 тоннеля:
![alt-текст](https://github.com/StuporMundiOmsk/OTUS_Networks/blob/main/Homeworks/05_EIGRP/R32_v6_routes_1.jpg "R32_v6_routes_1")


В конфигурацию R16 добавлен v6 префикс-лист, разрешающий умолчание и запрещающий по-умолчанию все остальное, затем префик лист это повешен на исходящие EIGRP апдейты в сторону R32:
```
!
ipv6 prefix-list Default seq 5 permit ::/0
!
!
 address-family ipv6 unicast autonomous-system 1
  !
  af-interface Ethernet0/1
   summary-address 2001:56:40::/48
  exit-af-interface
  !
  topology base
   distribute-list prefix-list Default out Ethernet0/3
  exit-af-topology
  eigrp router-id 16.16.16.16
 exit-address-family
!
```
Теперь, наконец-то, R32 окончательно и бесповортно ущемлен и со всех сторон закрепощен! Ура!!!
![alt-текст](https://github.com/StuporMundiOmsk/OTUS_Networks/blob/main/Homeworks/05_EIGRP/R32_v6_routes_2.jpg "R32_v6_routes_2")

 





