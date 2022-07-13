# NAT, DHCP, NTP

## Задание:
1. Настроить трансляцию трафика из Москау в один адрес
2. Настроить трансляцию трафика из Ленинграда в пул из пяти адресов
3. Настроить статическую трансляцию для R20
4. Настроить трансляцию для R19 так, чтобы он был доступен любому школьнику в Интернете
5. Настроить трансляцию трафика из Чокурдаха
6. Настроить DHCP в Москау (R12-13) и раздать адресов направо/налево
7. Настроить NTP в Москау (R12-13) и привести московитов к единообразию

## Схема:
![alt-текст](https://github.com/StuporMundiOmsk/OTUS_Networks/blob/main/Homeworks/06_BGP/Topology_Full.jpg "Итоговая картина")

## Решение:

## 1) NAT с перегрузкой на R14 и R15 настраивается абсолютно аналогично
### PAT в Москау:
```
!
interface Ethernet0/2
!
ip nat outside
!
!
interface Ethernet0/0
!
ip nat inside
!
!
interface Ethernet0/1
!
ip nat inside
!
!
interface Ethernet0/3
!
ip nat inside
!
!
interface Ethernet1/0
!
ip nat inside
!
!
ip nat pool 99 10.45.99.99 10.45.99.99 netmask 255.255.255.0
ip nat inside source list NAT pool 99 overload
!
ip access-list extended NAT
 permit ip 10.45.0.0 0.0.255.255 any
 deny   ip any any
!

```
![alt-текст](https://github.com/StuporMundiOmsk/OTUS_Networks/blob/main/Homeworks/07_NAT_DHCP_NTP/PAT_Moscow.jpg "PAT_Moscow") 


## 2) NAT с перегрузкой в пять адресов в Ленинграде настраивается весьма ординарно
### PAT в Ленинграде:
```
!
interface Ethernet0/2
!
ip nat outside
!
!
interface Ethernet0/3
!
ip nat outside
!
!
interface Ethernet0/0
!
ip nat inside
!
!
interface Ethernet0/1
!
ip nat inside
!
!
ip nat pool 99 10.40.99.99 10.40.99.103 netmask 255.255.255.0
ip nat inside source list NAT pool 99 overload
!
!
ip access-list extended NAT
 permit ip 10.40.0.0 0.0.255.255 any
 deny   ip any any
!
!
```
Хосты в Москау отказываются отвечать (PAT работает как файерволл, однако трансляция идет. На всякий пропинговал и дальний интерфейс Ламаса):
![alt-текст](https://github.com/StuporMundiOmsk/OTUS_Networks/blob/main/Homeworks/07_NAT_DHCP_NTP/PAT_Lenin.jpg "PAT_Lenin") 


## 3) Статическая трансляция для R20 настраивается на R14-15 весьма прозаически
```
!
ip nat inside source static 10.45.6.20 10.45.99.100
!
```
Вот так выглядят работающие одновременно на R15 PAT и static NAT:
![alt-текст](https://github.com/StuporMundiOmsk/OTUS_Networks/blob/main/Homeworks/07_NAT_DHCP_NTP/NAT_R20.jpg "NAT_R20") 

### Интересная ситуация! После добавления статики для R20 ему стали доступны оба хоста Ленинграда (после разворачивания ПАТов в обеих столицах связность хостов пропала, что логично), хотя никак явно на R18 он не прописан! Ну а сам R20 доступен по наружному адресу своему - здесь все ожидаемо. Как статический нат в Москве открыл дорогу через пат Ленинграда - вопрос, на который я пока ответа не нашел. 
![alt-текст](https://github.com/StuporMundiOmsk/OTUS_Networks/blob/main/Homeworks/07_NAT_DHCP_NTP/R20_Pheno.jpg "R20_Pheno") 


## 4) Вооруженные опытом статических трансляций из предыдущего задания, мы начинаем догадываться, что лучший способ открыть доступ через нат - задать статическую трансляцию без ограничений по портам. Однако, кажется, что в задании не случайно фигурирует "удаленное управление", поэтому сделаю статику для R19, но только для SSH. Настройки на R14-15 привычно выглядят одинаково.
```
!
ip nat inside source static tcp 10.45.1.19 22 10.45.99.56 22
!
```

## 5) Настройка ПАТ в Чокурдахе на R28, опять же, выглядит достаточно тривиально
```
!
interface Ethernet0/0
!
ip nat outside
!
!
interface Ethernet0/1
!
ip nat outside
!
!
interface Ethernet0/2
!
ip nat outside
!
!
!
ip nat pool 99 10.67.99.99 10.67.99.99 netmask 255.255.255.0
ip nat inside source list NAT pool 99 overload
!
!
ip access-list extended NAT
 permit ip 10.40.0.0 0.0.255.255 any
 deny   ip any any
!
!
```
![alt-текст](https://github.com/StuporMundiOmsk/OTUS_Networks/blob/main/Homeworks/07_NAT_DHCP_NTP/PAT_Choku.jpg "PAT_Choku") 


## 6) Настройка DHCP на обоих свитчах (именно свитчах, потому что у нас ведь везде L3 без всяких виланов, а значит DHCP на R12-13, помимо проблемы их синхронизации, вообще не будет работать корректно, т.к. отсутствует метка в виде тега вилана, на основании которого сервер должен понимать, какой пул использовать для ответа на запрос). DHCP опускается на уровень SW2-3 без релеев. В принципе, в начале курса была лаба по DHCP, т.ч. не критично.
### SW3:  
```
!
ip dhcp excluded-address 10.45.115.3

!
ip dhcp pool 115
 network 10.45.115.0 255.255.255.0
 default-router 10.45.115.3
!
```

### SW2:
```
!
ip dhcp excluded-address 10.45.116.2
!
!
ip dhcp pool 116
 network 10.45.116.0 255.255.255.0
 default-router 10.45.116.2
!
```
Хосты получили адреса:
![alt-текст](https://github.com/StuporMundiOmsk/OTUS_Networks/blob/main/Homeworks/07_NAT_DHCP_NTP/DHCP.jpg "DHCP") 


## 7) NTP уже никак не привязан к сетям, а потому его можно настроить вполне на R12-13 (настройки аналогичны). Однако вопрос их синхронизации все ещё стоит довольно остро.
### Сервера: ntp broadcast на всех интерфейсах, ntp master в глобальной конфигурации
### Клиенты: ntp server 10.45.2.12 и 3.13 (правильнее использовать лупбеки, но раз уж полная связность айпишная, то пусть будет физика) в глобальной конфигурации и ntp broadcast client на интерфейсах, смотрящих примерно в сторону серверов
![alt-текст](https://github.com/StuporMundiOmsk/OTUS_Networks/blob/main/Homeworks/07_NAT_DHCP_NTP/NTP.jpg "NTP") 



