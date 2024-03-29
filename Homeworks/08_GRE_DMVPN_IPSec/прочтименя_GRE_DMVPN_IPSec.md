﻿# GRE_DMVPN

## Задание:
1. Настроить GRE между двумя столицами
2. Настроить DMVPN между Златоглавой и Лабытнанги с Чокурдахом


## Схема:
![alt-текст](https://github.com/StuporMundiOmsk/OTUS_Networks/blob/main/Homeworks/Topology.jpg "Итоговая картина")

## Решение:

## 1) GRE (для более комфортной работы с туннелями были деактивированы PATы из предыдущей лабы, а провайдеры через BGP стали распространять друг другу сети внешних соединений, опять же, друг с другом, т.к. туннели устанавливаются на основе внешних адресов территорий)
### GRE в Москау (туннель привязан к интерфейсу, через который москвичи смотрят в мир при обычном режиме работы, добавлен специфический маршрут до Ленинграда через туннель):
```
!
interface Tunnel0
 ip address 10.56.56.15 255.255.255.0
 ip mtu 1400
 ip tcp adjust-mss 1360
 tunnel source 102.45.1.15
 tunnel destination 100.40.1.18
!
ip route 10.40.0.0 255.255.0.0 10.56.56.18
!

```
### GRE в Ленинграде (туннель привязан к интерфейсу, через который ленинградцы смотрят в мир при обычном режиме работы, добавлен специфический маршрут до Москау через туннель):
```
!
interface Tunnel0
 ip address 10.56.56.18 255.255.255.0
 ip mtu 1400
 ip tcp adjust-mss 1360
 tunnel source 100.40.1.18
 tunnel destination 102.45.1.15
!
ip route 10.45.0.0 255.255.0.0 10.56.56.15
!

```

### В итоге хосты в столицах видят другу друга вновь, а трассировка показывает, что делают они это через туннель

![alt-текст](https://github.com/StuporMundiOmsk/OTUS_Networks/blob/main/Homeworks/08_GRE_DMVPN_IPSec/GRE.jpg "GRE") 


## 2.1) GRE+NHRP 
### HUB в Москау (статика до внешних адресов споков+GRE+NHRP):
```
!
ip route 100.62.1.27 255.255.255.255 102.45.1.21
ip route 100.67.1.28 255.255.255.255 102.45.1.21
!
interface Tunnel1
 ip address 10.56.65.15 255.255.255.0
 no ip redirects
 ip mtu 1400
 ip nhrp map multicast dynamic
 ip nhrp network-id 1
 ip tcp adjust-mss 1360
 tunnel source Ethernet0/2
 tunnel mode gre multipoint
 tunnel key 1
!

```

### Spoke в Чокурдах (умолчание+GRE+NHRP):
```
!
ip route 0.0.0.0 0.0.0.0 100.67.1.25
!
interface Tunnel1
 ip address 10.56.65.28 255.255.255.0
 no ip redirects
 ip mtu 1400
 ip nhrp map 10.56.65.15 102.45.1.15
 ip nhrp map multicast 102.45.1.15
 ip nhrp network-id 1
 ip nhrp nhs 10.56.65.15
 ip tcp adjust-mss 1360
 tunnel source Ethernet0/1
 tunnel mode gre multipoint
 tunnel key 1
!

```

### Spoke в Лабытнанги (GRE+NHRP):
```
!
interface Tunnel1
 ip address 10.56.65.27 255.255.255.0
 no ip redirects
 ip mtu 1400
 ip nhrp map 10.56.65.15 102.45.1.15
 ip nhrp map multicast 102.45.1.15
 ip nhrp network-id 1
 ip nhrp nhs 10.56.65.15
 ip tcp adjust-mss 1360
 tunnel source Ethernet0/0
 tunnel mode gre multipoint
 tunnel key 1
!

```

### На хабе в Москау подняты туннели:
![alt-текст](https://github.com/StuporMundiOmsk/OTUS_Networks/blob/main/Homeworks/08_GRE_DMVPN_IPSec/R15_GRE%2BNHRP.jpg "R15_GRE+NHRP") 

### Споки ноки:
![alt-текст](https://github.com/StuporMundiOmsk/OTUS_Networks/blob/main/Homeworks/08_GRE_DMVPN_IPSec/R27_GRE%2BNHRP.jpg "R27_GRE+NHRP") 
![alt-текст](https://github.com/StuporMundiOmsk/OTUS_Networks/blob/main/Homeworks/08_GRE_DMVPN_IPSec/R28_GRE%2BNHRP.jpg "R28_GRE+NHRP") 


## 2.2) EIGRP+DMVPN
### EIGRP на хабе и споках (аналогично) включается для сетей mGRE и сетей площадок, отключается автосуммирование; на хабе отключается расщепление горизонта и подстановка себя в качестве некстхопа
```
!
router eigrp 1
 network 10.45.0.0 0.0.255.255
 network 10.56.65.0 0.0.0.255
!
!
interface Tunnel1
! 
 no ip next-hop-self eigrp 1
 no ip split-horizon eigrp 1
!
```

### EIGRP поднялось, сеть R27 (пришлось поднять лупбек) видна в таблице напрямую, а не через хаб; трассировка наглядно показывает механизм DMVPN во 2-ой фазе
![alt-текст](https://github.com/StuporMundiOmsk/OTUS_Networks/blob/main/Homeworks/08_GRE_DMVPN_IPSec/R15_EIGRP.jpg "R15_EIGRP")
![alt-текст](https://github.com/StuporMundiOmsk/OTUS_Networks/blob/main/Homeworks/08_GRE_DMVPN_IPSec/R28_EIGRP.jpg "R28_EIGRP")  


# GRE&DMVPN over IPSec

## Задание:
1. Покрыть шифрованием существующие DMVPN
2. Покрыть шифрованием существующие GRE
3. Исправить шифрование GRE
4. Исправить шифрование DMVPN   

## 1.1) IPSec Phase 1, CA

### IPSec Phase 1
```
!
crypto isakmp policy 10
 encr aes 192
 hash sha256
 group 2
!

```

### CA на R15
![alt-текст](https://github.com/StuporMundiOmsk/OTUS_Networks/blob/main/Homeworks/08_GRE_DMVPN_IPSec/R15_RSA_1.jpg "R15_RSA_1")
![alt-текст](https://github.com/StuporMundiOmsk/OTUS_Networks/blob/main/Homeworks/08_GRE_DMVPN_IPSec/R15_RSA_2.jpg "R15_RSA_2")
![alt-текст](https://github.com/StuporMundiOmsk/OTUS_Networks/blob/main/Homeworks/08_GRE_DMVPN_IPSec/R15_PKI.jpg "R15_PKI")  

### Получение ключей на споке и хабе
![alt-текст](https://github.com/StuporMundiOmsk/OTUS_Networks/blob/main/Homeworks/08_GRE_DMVPN_IPSec/R27_Cert.jpg "R27_Cert")  
![alt-текст](https://github.com/StuporMundiOmsk/OTUS_Networks/blob/main/Homeworks/08_GRE_DMVPN_IPSec/R15_Cert.jpg "R15_Cert") 


## 1.2) IPSEc Phase 2 (Totally failed)

### Как и сказано в дисклеймере, полнейший провал (при аналогичных настройках хаба и споков) - не сильный защитник, нуждаюсь в помощи!
![alt-текст](https://github.com/StuporMundiOmsk/OTUS_Networks/blob/main/Homeworks/08_GRE_DMVPN_IPSec/Failed%20on%20Spoke.jpg "Failed on Spoke")  
![alt-текст](https://github.com/StuporMundiOmsk/OTUS_Networks/blob/main/Homeworks/08_GRE_DMVPN_IPSec/Failed%20on%20Hub.jpg "Failed on Hub") 

### Ключики на R15
```
!
crypto pki certificate chain R15
 certificate ca 01
  30820170 3082011A A0030201 02020101 300D0609 2A864886 F70D0101 04050030
  0E310C30 0A060355 04031303 52313530 1E170D32 32303732 31303832 3935315A
  170D3235 30373230 30383239 35315A30 0E310C30 0A060355 04031303 52313530
  5C300D06 092A8648 86F70D01 01010500 034B0030 48024100 A6954DFF 3ECC2320
  428BAEFA 7A0A0D3C 29EFFA7B CCC95E18 F56D5E35 24F6F663 E420DFA8 FB0752E4
  5C133362 B8AA20A0 961427FB 4A7F2A14 8CC394DE 1B7F035F 02030100 01A36330
  61300F06 03551D13 0101FF04 05300301 01FF300E 0603551D 0F0101FF 04040302
  0186301F 0603551D 23041830 168014F6 8138200C 4BC52116 4D1F5F10 B618D391
  90C39530 1D060355 1D0E0416 0414F681 38200C4B C521164D 1F5F10B6 18D39190
  C395300D 06092A86 4886F70D 01010405 00034100 4CCAA6BF E91DC6A4 9E711AED
  18661BC7 2191B014 5EB523FA 58402497 B72DD89D BAF76F3B 10F48383 7FD06787
  85FC26D7 49A2CDD2 2854F17E 29CC8E65 3CD52458
        quit
crypto pki certificate chain SELF
 certificate 04
  3082017F 30820129 A0030201 02020104 300D0609 2A864886 F70D0101 05050030
  0E310C30 0A060355 04031303 52313530 1E170D32 32303732 31303930 3532375A
  170D3233 30373231 30393035 32375A30 31312F30 0F060355 04051308 36373130
  39313034 301C0609 2A864886 F70D0109 02160F52 31352E6F 7475736C 61622E63
  6F6D305C 300D0609 2A864886 F70D0101 01050003 4B003048 0241009E FE6E5E94
  5EAD79ED EBE0F69F 45358972 CA4E589D 79A33D0A B4763D1B 10EF8779 4DA21C80
  3098F96B EE9E4570 21BF4564 C299A473 13CA29CE DAF14E4F E0D6BB02 03010001
  A34F304D 300B0603 551D0F04 04030205 A0301F06 03551D23 04183016 8014F681
  38200C4B C521164D 1F5F10B6 18D39190 C395301D 0603551D 0E041604 1423FFD2
  6E6EAD5C A741BC6E 6E2C2462 208904CF EF300D06 092A8648 86F70D01 01050500
  03410046 68E483AE A32F90ED B381831D CD0D1A02 16184C24 B09C124B 4BED7F7F
  C741BD4B 6A1B026F EBBD9F92 F9EDE18D 0F150DBA F9623682 02C16F2D 32DC7142 3172F9
        quit
 certificate ca 01
  30820170 3082011A A0030201 02020101 300D0609 2A864886 F70D0101 04050030
  0E310C30 0A060355 04031303 52313530 1E170D32 32303732 31303832 3935315A
  170D3235 30373230 30383239 35315A30 0E310C30 0A060355 04031303 52313530
  5C300D06 092A8648 86F70D01 01010500 034B0030 48024100 A6954DFF 3ECC2320
  428BAEFA 7A0A0D3C 29EFFA7B CCC95E18 F56D5E35 24F6F663 E420DFA8 FB0752E4
  5C133362 B8AA20A0 961427FB 4A7F2A14 8CC394DE 1B7F035F 02030100 01A36330
  61300F06 03551D13 0101FF04 05300301 01FF300E 0603551D 0F0101FF 04040302
  0186301F 0603551D 23041830 168014F6 8138200C 4BC52116 4D1F5F10 B618D391
  90C39530 1D060355 1D0E0416 0414F681 38200C4B C521164D 1F5F10B6 18D39190
  C395300D 06092A86 4886F70D 01010405 00034100 4CCAA6BF E91DC6A4 9E711AED
  18661BC7 2191B014 5EB523FA 58402497 B72DD89D BAF76F3B 10F48383 7FD06787
  85FC26D7 49A2CDD2 2854F17E 29CC8E65 3CD52458
        quit
!

```

### Ключики на R27
```
!
crypto pki certificate chain R15
 certificate 02
  3082017F 30820129 A0030201 02020102 300D0609 2A864886 F70D0101 05050030
  0E310C30 0A060355 04031303 52313530 1E170D32 32303732 31303834 3833315A
  170D3233 30373231 30383438 33315A30 31312F30 0F060355 04051308 36373130
  39323936 301C0609 2A864886 F70D0109 02160F52 32372E6F 7475736C 61622E63
  6F6D305C 300D0609 2A864886 F70D0101 01050003 4B003048 024100C5 F973669D
  AE33FA58 7C9DA12F ABD80EE0 4E389C86 0FE6F939 33854D21 65B0FCBC 104AEE4C
  D306DB44 71734663 A25292D7 469289F1 ADAC5053 08F601BF C56E1902 03010001
  A34F304D 300B0603 551D0F04 04030205 A0301F06 03551D23 04183016 8014F681
  38200C4B C521164D 1F5F10B6 18D39190 C395301D 0603551D 0E041604 14DC2638
  DDF62488 FB5C18B6 6A75F1BC 6313A169 B9300D06 092A8648 86F70D01 01050500
  03410035 44B72DCB BD913A33 27949057 69789F96 A808E91F 55953BF5 B650D632
  E6F087DC 81C4884A EB0BF21C D94E8E81 052E2B99 9040FD18 14C4A9CD E3719405 65C9AF
        quit
 certificate ca 01
  30820170 3082011A A0030201 02020101 300D0609 2A864886 F70D0101 04050030
  0E310C30 0A060355 04031303 52313530 1E170D32 32303732 31303832 3935315A
  170D3235 30373230 30383239 35315A30 0E310C30 0A060355 04031303 52313530
  5C300D06 092A8648 86F70D01 01010500 034B0030 48024100 A6954DFF 3ECC2320
  428BAEFA 7A0A0D3C 29EFFA7B CCC95E18 F56D5E35 24F6F663 E420DFA8 FB0752E4
  5C133362 B8AA20A0 961427FB 4A7F2A14 8CC394DE 1B7F035F 02030100 01A36330
  61300F06 03551D13 0101FF04 05300301 01FF300E 0603551D 0F0101FF 04040302
  0186301F 0603551D 23041830 168014F6 8138200C 4BC52116 4D1F5F10 B618D391
  90C39530 1D060355 1D0E0416 0414F681 38200C4B C521164D 1F5F10B6 18D39190
  C395300D 06092A86 4886F70D 01010405 00034100 4CCAA6BF E91DC6A4 9E711AED
  18661BC7 2191B014 5EB523FA 58402497 B72DD89D BAF76F3B 10F48383 7FD06787
  85FC26D7 49A2CDD2 2854F17E 29CC8E65 3CD52458
        quit
!

```

### Ключики на R28
```
!
crypto pki certificate chain R15
 certificate 03
  3082017F 30820129 A0030201 02020103 300D0609 2A864886 F70D0101 05050030
  0E310C30 0A060355 04031303 52313530 1E170D32 32303732 31303835 3534395A
  170D3233 30373231 30383535 34395A30 31312F30 0F060355 04051308 36373130
  39333132 301C0609 2A864886 F70D0109 02160F52 32382E6F 7475736C 61622E63
  6F6D305C 300D0609 2A864886 F70D0101 01050003 4B003048 024100B7 546F7D45
  167A3781 2E581177 8EFC4FF4 ED6BF2E3 DE15FC59 3DD4743D 5012922B B4D87F63
  B389D9E8 DDD4E337 1B084DEF 66DFC775 33B3AE5A 9B661319 07045702 03010001
  A34F304D 300B0603 551D0F04 04030205 A0301F06 03551D23 04183016 8014F681
  38200C4B C521164D 1F5F10B6 18D39190 C395301D 0603551D 0E041604 148EAC67
  E442E7DC 3AB26EF6 1092E203 2E00005E C5300D06 092A8648 86F70D01 01050500
  0341000E 0E2BE878 B08DC644 D904ACEC A4F2625E 7740098A A3B46082 D6E634C1
  0339E3CD 5EB57A9C F58BD7A2 4E1B1D69 AF753977 1CB3C9D3 1AE7E333 1FE24D51 931DFB
        quit
 certificate ca 01
  30820170 3082011A A0030201 02020101 300D0609 2A864886 F70D0101 04050030
  0E310C30 0A060355 04031303 52313530 1E170D32 32303732 31303832 3935315A
  170D3235 30373230 30383239 35315A30 0E310C30 0A060355 04031303 52313530
  5C300D06 092A8648 86F70D01 01010500 034B0030 48024100 A6954DFF 3ECC2320
  428BAEFA 7A0A0D3C 29EFFA7B CCC95E18 F56D5E35 24F6F663 E420DFA8 FB0752E4
  5C133362 B8AA20A0 961427FB 4A7F2A14 8CC394DE 1B7F035F 02030100 01A36330
  61300F06 03551D13 0101FF04 05300301 01FF300E 0603551D 0F0101FF 04040302
  0186301F 0603551D 23041830 168014F6 8138200C 4BC52116 4D1F5F10 B618D391
  90C39530 1D060355 1D0E0416 0414F681 38200C4B C521164D 1F5F10B6 18D39190
  C395300D 06092A86 4886F70D 01010405 00034100 4CCAA6BF E91DC6A4 9E711AED
  18661BC7 2191B014 5EB523FA 58402497 B72DD89D BAF76F3B 10F48383 7FD06787
  85FC26D7 49A2CDD2 2854F17E 29CC8E65 3CD52458
        quit
!

```

## 2.1) IPSec Phase 1, Cert (испуганный предыдущим горьким опытом, пробовал поднять на минималках)

### IPSec Phase 1 (R15 и R18 настраиваются аналогично)
```
!
crypto isakmp policy 20
 encr 3des
!
crypto ipsec transform-set GRE esp-aes esp-sha-hmac
 mode transport
!
!
access-list 101 permit gre host 102.45.1.15 host 100.40.1.18
!
!
crypto map MAP 10 ipsec-isakmp
 set peer 100.40.1.18
 set transform-set GRE
 match address 101
!
!
interface Ethernet0/2
!
 crypto map MAP
!

```

### R18 настроен аналогично R27-28 из предыдущих попыток, сертификат получил
![alt-текст](https://github.com/StuporMundiOmsk/OTUS_Networks/blob/main/Homeworks/08_GRE_DMVPN_IPSec/R18_Cert.jpg "R18_Cert")

### Однако, в лучших традициях, опять эпик фейл
![alt-текст](https://github.com/StuporMundiOmsk/OTUS_Networks/blob/main/Homeworks/08_GRE_DMVPN_IPSec/Failed%20GRE_IPSec.jpg "Failed GRE_IPSec") 


## Очень тянет сделать вывод о том, что IPSec - приспособление инквизиции для эффективного причинения боли! Но, скорее всего, сказывается нулевой опыт мой в секурность... 

## 3.1) На R15 и R18 применяется простейший IPSec с аутентификацией по паролю (разумеется, cisco)
![alt-текст](https://github.com/StuporMundiOmsk/OTUS_Networks/blob/main/Homeworks/08_GRE_DMVPN_IPSec/GRE_IPSec_preshare.jpg "GRE_IPSec_preshare")

## 3.2) Последовательность команд на R15/R18
```
R15:
crypto key generate rsa general-keys label R15 exportable modulus 2048
crypto pki server R15 
 no shutdown

R18:
crypto key generate rsa label R18 modulus 2048
crypto pki trustpoint R18
 enrollment url http://R15
 subject-name CN=R18,OU=STUPOR,O=OMSK,C=RU
 rsakeypair R18
 revocation-check none
crypto pki authenticate R18
crypto pki enroll R18

R15:
crypto pki server R15 grant all

R15/18:
crypto isakmp policy 20
 authentication rsa-sig

crypto ipsec transform-set GRE esp-aes esp-sha-hmac
crypto ipsec profile GRE
 set transform-set GRE

interface tunnel 0
 tunnel protection ipsec profile GRE

R15:
crypto pki trustpoint SELF
 enrollment url http://R15
 subject-name CN=R15,OU=STUPOR,O=OMSK,C=RU
 rsakeypair SELF
 revocation-check none
crypto pki authenticate SELF
crypto pki enroll SELF 

```

Всю настройку списывал с итоговой лабы, где все завелось - у меня же ничего не завелось. По той же схеме: ключики получаются, IPSec не поднимается.
![alt-текст](https://github.com/StuporMundiOmsk/OTUS_Networks/blob/main/Homeworks/08_GRE_DMVPN_IPSec/GRE_IPSec_CA.jpg "GRE_IPSec_CA")
