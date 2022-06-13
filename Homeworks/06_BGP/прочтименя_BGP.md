# BGP, часть 1 - EBGP 

## Задание:
1. Настроить EBGP-соседей на всех стыках Москау и Ленинграда с провайдерами
2. Устранить досадные мелочи, мешающие простым гражданам Ленинграда пинговать коренных московитов 

## Схема:
![alt-текст](https://github.com/StuporMundiOmsk/OTUS_Networks/blob/main/Homeworks/06_BGP/Topology_Full.jpg "Итоговая картина")

## Решение:

## 1) Настройка EBGP по всему маршруту следования Радищева (в столицах для объединения перфиксов создаются маршруты в никуда - в той, которая северная, такой подход чуть позже аукнется)
### R14 (Москау_1):
```
!
router bgp 1001
 bgp router-id 14.14.14.14
 bgp log-neighbor-changes
 network 10.45.0.0 mask 255.255.0.0
 neighbor 101.45.1.22 remote-as 101
!
ip route 10.45.0.0 255.255.0.0 Null0

```
### R15 (Москау_2):
```
!
router bgp 1001
 bgp router-id 15.15.15.15
 bgp log-neighbor-changes
 network 10.45.0.0 mask 255.255.0.0
 neighbor 102.45.1.21 remote-as 301
!
ip route 10.45.0.0 255.255.0.0 Null0

```  
### R22 (Киторн против всех):
``` 
!
router bgp 101
 bgp router-id 22.22.22.22
 bgp log-neighbor-changes
 neighbor 100.101.1.23 remote-as 520
 neighbor 101.45.1.14 remote-as 1001
 neighbor 101.102.1.21 remote-as 301
!

```  
### R21 (Ламас против всех):
``` 
!
router bgp 301
 bgp router-id 21.21.21.21
 bgp log-neighbor-changes
 neighbor 100.102.1.24 remote-as 520
 neighbor 101.102.1.22 remote-as 101
 neighbor 102.45.1.15 remote-as 1001
!

``` 
### R23 (Триада в сторону Киторна):
``` 
!
router bgp 520
 bgp router-id 23.23.23.23
 bgp log-neighbor-changes
 neighbor 100.101.1.22 remote-as 101
!

```
### R24 (Триада в сторону Ламаса и Ленинграда_1):
``` 
!
router bgp 520
 bgp router-id 24.24.24.24
 bgp log-neighbor-changes
 neighbor 100.40.1.18 remote-as 2042
 neighbor 100.102.1.21 remote-as 301
!

```
### R26 (Триада в сторону Ленинграда_2):
``` 
!
router bgp 520
 bgp router-id 26.26.26.26
 bgp log-neighbor-changes
 neighbor 100.40.2.18 remote-as 2042
!

```
### R18 (Ленинград в Европу и Азию):
``` 
!
router bgp 2042
 bgp router-id 18.18.18.18
 bgp log-neighbor-changes
 network 10.40.0.0 mask 255.255.0.0
 neighbor 100.40.1.24 remote-as 520
 neighbor 100.40.2.26 remote-as 520
!
ip route 10.40.0.0 255.255.0.0 Null0

```

## 2) Внутри Триады образовались технологические отверстия в BGP, поэтому, чтобы добиться связности между столицами, пришлось изничтожить маршрут по умолчанию на R14 (временно! он би бэк сун), чтобы путь Радищева приобрел простые и понятные очертания: R15-R21-R24-R18.
Но неприятности на этом не закончились. В предыдущей домашке внутри Ленинграда в рамках EIGRP было настроено суммирование маршрутов до R18, всю красоту которого разрушил блекхольный маршрут, необходимый для BGP. Суммаризацию на R16 и R17 пришлось свернуть.
Маршруты на границах городов:
![alt-текст](https://github.com/StuporMundiOmsk/OTUS_Networks/blob/main/Homeworks/06_BGP/R15%26R18_BGP.jpg "R15&R18_BGP") 
Удачные переговоры лучших представителей двух городов:
![alt-текст](https://github.com/StuporMundiOmsk/OTUS_Networks/blob/main/Homeworks/06_BGP/Pongs_1.jpg "Pongs_1")   


# BGP, часть 2 - IBGP 

## Задание:
1. Настроить IBGP в Москау и Триаде
2. Москву приоритетно выпускать через Ламас. Ленинградцев принимать в Триады сразу через два окна.
3. Убедиться, что нововведения не помешали простым гражданам Ленинграда пинговать коренных московитов 

## Решение:
## 1) Добавленные соседи в Москау и Триаде (один раз даже используется прогрессивная группа!)
### R14:
``` 
neighbor 10.45.20.15 remote-as 1001
neighbor 10.45.20.15 next-hop-self

```
### R15:
``` 
neighbor 10.45.20.14 remote-as 1001
neighbor 10.45.20.14 next-hop-self

```
### R23:
``` 
neighbor 10.100.1.25 remote-as 520
neighbor 10.100.1.25 next-hop-self
neighbor 10.100.2.24 remote-as 520
neighbor 10.100.2.24 next-hop-self
neighbor 10.100.3.26 remote-as 520
neighbor 10.100.3.26 next-hop-self

```
### R24:
``` 
neighbor 10.100.1.25 remote-as 520
neighbor 10.100.1.25 next-hop-self
neighbor 10.100.2.23 remote-as 520
neighbor 10.100.2.23 next-hop-self
neighbor 10.100.3.26 remote-as 520
neighbor 10.100.3.26 next-hop-self

```
### R25:
``` 
neighbor 10.100.1.23 remote-as 520
neighbor 10.100.1.23 next-hop-self
neighbor 10.100.2.24 remote-as 520
neighbor 10.100.2.24 next-hop-self
neighbor 10.100.4.26 remote-as 520
neighbor 10.100.4.26 next-hop-self

```
### R26:
``` 
neighbor IBGP peer-group
neighbor IBGP remote-as 520
neighbor IBGP next-hop-self
neighbor 10.100.2.23 peer-group IBGP
neighbor 10.100.3.24 peer-group IBGP
neighbor 10.100.4.25 peer-group IBGP
```

## 2) Чтобы сменить приоритеты Москау в пользу Ламаса, принято решение всем префиксам от Киторна удлинять As Path (и да, вот сейчас уже можно вернуть одинаковые маршруты по умолчанию в сторону провайдеров на R14 и R15). В Ленинграде применена строчка, разрешающая балансировку.
### R14:
``` 
!
route-map AS-PREP permit 10
 set as-path prepend 1001 1001 1001
!
neighbor 101.45.1.22 route-map AS-PREP in

```
![alt-текст](https://github.com/StuporMundiOmsk/OTUS_Networks/blob/main/Homeworks/06_BGP/AS-PREP.jpg "AS-PREP") 
### R18:
``` 
!
router bgp 2042
!
maximum-paths 2
```
![alt-текст](https://github.com/StuporMundiOmsk/OTUS_Networks/blob/main/Homeworks/06_BGP/MAX-PATHS.jpg "MAX-PATHS") 

## 3) Вновь удачные переговоры лучших представителей двух городов:
![alt-текст](https://github.com/StuporMundiOmsk/OTUS_Networks/blob/main/Homeworks/06_BGP/Pongs_2.jpg "Pongs_2") 


# BGP, часть 3 - Filtering 

## Задание:
1) Отфильтровать "транзитный" трафик в Москау, используя AS-Path фильтры
2) Отфильтровать "транзитный" трафик в Ленинграде, используя префикс-листы
3) Передавать из Киторна в Москау только маршрут по умолчанию
4) Передавать из Ламаса в Москау только маршрут по умолчанию и префиксы Ленинграда
5) Убедиться, что связность не пострадала  

## Решение:
## 1) Под "транзитным" трафиком решено понимать все, что зародилось где-то кроме известных нам AS.
### R14:
``` 
!
ip as-path access-list 10 permit _2042$
ip as-path access-list 10 permit _101$
ip as-path access-list 10 permit _301$
ip as-path access-list 10 permit _520$
ip as-path access-list 10 deny .*
!
router bgp 1001
!
neighbor 101.45.1.22 filter-list 10 in
```
### R15:
``` 
!
ip as-path access-list 10 permit _2042$ | _520$ | _301$ | _101$
ip as-path access-list 10 deny .*
!
router bgp 1001
!
neighbor 102.45.1.21 filter-list 10 in
```

## 2) В контексте префикс-листов было принято решение понимать транзитными те префиксы, которые не имеют отношения к известным нам.
### R18:
``` 
!
ip prefix-list PL_IN seq 5 permit 10.0.0.0/8 ge 16 le 24
ip prefix-list PL_IN seq 10 permit 100.0.0.0/8 ge 16 le 24
ip prefix-list PL_IN seq 15 permit 101.0.0.0/8 ge 16 le 24
ip prefix-list PL_IN seq 20 permit 102.0.0.0/8 ge 16 le 24
ip prefix-list PL_IN seq 25 deny 0.0.0.0/0 le 32
!
router bgp 1001
!
neighbor 100.40.1.24 prefix-list PL_IN in
!
neighbor 100.40.2.26 prefix-list PL_IN in
```

## 3) Из Киторна в Москау должно прилетать только умолчание (да, ручное умолчание опять пришлось удалять)
### R22:
```
!
router bgp 101
!
neighbor 101.45.1.14 default-originate
neighbor 101.45.1.14 prefix-list PL_OUT out
!
ip prefix-list PL_OUT seq 5 deny 0.0.0.0/0 le 32
``` 
![alt-текст](https://github.com/StuporMundiOmsk/OTUS_Networks/blob/main/Homeworks/06_BGP/R14_BGP.jpg "R14_BGP") 

## 4) Из Ламаса в Москау должно прилетать только умолчание и префикс Ленинграда (да, ручное умолчание опять пришлось удалять)
### R21 (R15 начал адекватно воспринимать маршруты только после удаления AS-Path ACL из предыдущего подзадания):
```
!
router bgp 101
!
neighbor 101.45.1.14 default-originate
neighbor 101.45.1.14 prefix-list PL_OUT out
!
!
ip prefix-list PL_OUT seq 5 permit 10.0.0.0/8 ge 16 le 24
ip prefix-list PL_OUT seq 10 deny 0.0.0.0/0 le 32
!
``` 
![alt-текст](https://github.com/StuporMundiOmsk/OTUS_Networks/blob/main/Homeworks/06_BGP/R21_advertised.jpg "R21_advertised")
![alt-текст](https://github.com/StuporMundiOmsk/OTUS_Networks/blob/main/Homeworks/06_BGP/R15_BGP.jpg "R15_BGP")

## 5) Специально выбраны другие писюки, чтобы не создавалось в этих многочисленных пингах впечатление обмана!
![alt-текст](https://github.com/StuporMundiOmsk/OTUS_Networks/blob/main/Homeworks/06_BGP/Pongs_3.jpg "Pongs_3") 
