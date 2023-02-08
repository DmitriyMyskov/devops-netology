# Домашнее задание к занятию "3.6. Компьютерные сети, лекция 1"

***
## Работа c HTTP через телнет.
- Подключитесь утилитой телнет к сайту stackoverflow.com
`telnet stackoverflow.com 80`
- Отправьте HTTP запрос
```bash
GET /questions HTTP/1.0
HOST: stackoverflow.com
```
###
```bash
dmitriy@dmitriy-lin:~$ telnet stackoverflow.com 80
Trying 151.101.1.69...
Connected to stackoverflow.com.
Escape character is '^]'.
GET /questions HTTP/1.0
HOST: stackoverflow.com

HTTP/1.1 403 Forbidden
Connection: close
Content-Length: 1916
Server: Varnish
Retry-After: 0
Content-Type: text/html
Accept-Ranges: bytes
Date: Wed, 08 Feb 2023 09:29:36 GMT
Via: 1.1 varnish
X-Served-By: cache-fra-eddf8230136-FRA
X-Cache: MISS
X-Cache-Hits: 0
X-Timer: S1675848577.698252,VS0,VE1
X-DNS-Prefetch-Control: off
```
403 Forbidden — стандартный код ответа HTTP, означающий, что доступ к запрошенному ресурсу запрещен. Сервер понял запрос, но не выполнит его.

***
## Повторите задание 1 в браузере, используя консоль разработчика F12.
- откройте вкладку `Network`
- отправьте запрос http://stackoverflow.com
- найдите первый ответ HTTP сервера, откройте вкладку `Headers`
- укажите в ответе полученный HTTP код
- проверьте время загрузки страницы, какой запрос обрабатывался дольше всего?
- приложите скриншот консоли браузера в ответ.
###
Первый ответ 200 - указывает, что запрос выполнен успешно.
Общее время загрузки:
* Запросы: 49
* Перенесено: 51.7 kB.
* Ресурсы: 2.4 MB
* Завершено: 6.36 с.
* DOMContentLoaded: 940 мс.
* Загрузить: 1.38 с.
Дольше всего обрабатывался запрос GET https://stackoverflow.com/ 348мс

![DZ-3-6-1.png](images/DZ-3-6-1.png)
![DZ-3-6-2.png](images/DZ-3-6-2.png)
![DZ-3-6-3.png](images/DZ-3-6-3.png)

***
## Какой IP адрес у вас в интернете?
###
```bash
dmitriy@dmitriy-lin:~$ wget -qO- eth0.me
158.46.80.100
```

***
## Какому провайдеру принадлежит ваш IP адрес? Какой автономной системе AS? Воспользуйтесь утилитой `whois`
###
E-Light-Telecom Ltd. | AS39927

```bash
dmitriy@dmitriy-lin:~$ whois 158.46.80.100

#
# ARIN WHOIS data and services are subject to the Terms of Use
# available at: https://www.arin.net/resources/registry/whois/tou/
#
# If you see inaccuracies in the results, please report at
# https://www.arin.net/resources/registry/whois/inaccuracy_reporting/
#
# Copyright 1997-2023, American Registry for Internet Numbers, Ltd.
#


NetRange:       158.46.0.0 - 158.46.255.255
CIDR:           158.46.0.0/16
NetName:        RIPE-ERX-158-46-0-0
NetHandle:      NET-158-46-0-0-1
Parent:         NET158 (NET-158-0-0-0-0)
NetType:        Early Registrations, Transferred to RIPE NCC
OriginAS:       
Organization:   RIPE Network Coordination Centre (RIPE)
RegDate:        2010-11-03
Updated:        2010-11-17
Comment:        These addresses have been further assigned to users in
Comment:        the RIPE NCC region. Contact information can be found in
Comment:        the RIPE database at http://www.ripe.net/whois
Ref:            https://rdap.arin.net/registry/ip/158.46.0.0

ResourceLink:  https://apps.db.ripe.net/search/query.html
ResourceLink:  whois.ripe.net


OrgName:        RIPE Network Coordination Centre
OrgId:          RIPE
Address:        P.O. Box 10096
City:           Amsterdam
StateProv:      
PostalCode:     1001EB
Country:        NL
RegDate:        
Updated:        2013-07-29
Ref:            https://rdap.arin.net/registry/entity/RIPE

ReferralServer:  whois://whois.ripe.net
ResourceLink:  https://apps.db.ripe.net/search/query.html

OrgAbuseHandle: ABUSE3850-ARIN
OrgAbuseName:   Abuse Contact
OrgAbusePhone:  +31205354444 
OrgAbuseEmail:  abuse@ripe.net
OrgAbuseRef:    https://rdap.arin.net/registry/entity/ABUSE3850-ARIN

OrgTechHandle: RNO29-ARIN
OrgTechName:   RIPE NCC Operations
OrgTechPhone:  +31 20 535 4444 
OrgTechEmail:  hostmaster@ripe.net
OrgTechRef:    https://rdap.arin.net/registry/entity/RNO29-ARIN


#
# ARIN WHOIS data and services are subject to the Terms of Use
# available at: https://www.arin.net/resources/registry/whois/tou/
#
# If you see inaccuracies in the results, please report at
# https://www.arin.net/resources/registry/whois/inaccuracy_reporting/
#
# Copyright 1997-2023, American Registry for Internet Numbers, Ltd.
#



Найдено перенаправление на whois.ripe.net.

% This is the RIPE Database query service.
% The objects are in RPSL format.
%
% The RIPE Database is subject to Terms and Conditions.
% See http://www.ripe.net/db/support/db-terms-conditions.pdf

% Note: this output has been filtered.
%       To receive output for a database update, use the "-B" flag.

% Information related to '158.46.0.0 - 158.46.127.255'

% Abuse contact for '158.46.0.0 - 158.46.127.255' is 'abuse@eltc.ru'

inetnum:        158.46.0.0 - 158.46.127.255
netname:        GOODLINE-INFO
descr:          E-Light-Telecom
descr:          Novokuznetsk city
country:        RU
org:            ORG-EA385-RIPE
admin-c:        KK7315-RIPE
tech-c:         KS10342-RIPE
status:         ASSIGNED PA
mnt-by:         ELT-MNT
mnt-lower:      ELT-MNT
mnt-routes:     ELT-MNT
mnt-domains:    ELT-MNT
created:        2012-10-12T08:46:46Z
last-modified:  2018-11-15T07:15:04Z
source:         RIPE # Filtered

organisation:   ORG-EA385-RIPE
org-name:       E-Light-Telecom Ltd.
country:        RU
org-type:       LIR
address:        Kuznetsky 18
address:        650000
address:        Kemerovo
address:        RUSSIAN FEDERATION
phone:          +73842452999
fax-no:         +73842452999
abuse-c:        AR16650-RIPE
admin-c:        KK7315-RIPE
tech-c:         KS10342-RIPE
tech-c:         OE528-RIPE
mnt-ref:        ELT-MNT
mnt-ref:        RIPE-NCC-HM-MNT
mnt-by:         RIPE-NCC-HM-MNT
mnt-by:         ELT-MNT
created:        2008-09-15T12:23:31Z
last-modified:  2020-12-16T13:23:27Z
source:         RIPE # Filtered

person:         Konstantin Karavaev
address:        Russian Federation
address:        Kemerovo
address:        650099 Kuznetsky 18
org:            ORG-EA385-RIPE
phone:          +73842452999
phone:          +73842452893
nic-hdl:        KK7315-RIPE
mnt-by:         ELT-MNT
created:        2018-11-14T03:45:11Z
last-modified:  2018-11-14T03:53:45Z
source:         RIPE # Filtered

person:         Konstantin Shchukin
address:        Russian Federation
address:        Kemerovo
address:        650099 Kuznetsky 18
org:            ORG-EA385-RIPE
phone:          +73842452999
phone:          +73842452893
nic-hdl:        KS10342-RIPE
mnt-by:         ELT-MNT
created:        2018-11-14T03:48:38Z
last-modified:  2018-11-14T03:51:40Z
source:         RIPE # Filtered

% Information related to '158.46.0.0/17AS39927'

route:          158.46.0.0/17
descr:          Goodline.info
descr:          Novokuznetsk, Russia
descr:          RU
origin:         AS39927
mnt-by:         ELT-MNT
created:        2016-05-04T09:34:30Z
last-modified:  2016-05-04T09:34:30Z
source:         RIPE

% This query was served by the RIPE Database Query Service version 1.105 (ABERDEEN)
```

***
## Через какие сети проходит пакет, отправленный с вашего компьютера на адрес 8.8.8.8? Через какие AS? Воспользуйтесь утилитой `traceroute`
###
```bash
dmitriy@dmitriy-lin:~$ traceroute -An 8.8.8.8
traceroute to 8.8.8.8 (8.8.8.8), 30 hops max, 60 byte packets
 1  192.168.10.1 [*]  0.607 ms  0.558 ms  0.529 ms
 2  10.200.22.1 [*]  43.431 ms  43.400 ms  43.375 ms
 3  10.77.77.10 [*]  82.975 ms  82.894 ms  84.451 ms
 4  46.246.96.1 [AS42708]  85.946 ms  86.057 ms  86.035 ms
 5  80.67.3.188 [AS42708]  85.101 ms  85.065 ms  85.256 ms
 6  80.67.4.156 [AS42708]  86.122 ms 80.67.4.154 [AS42708]  83.107 ms  83.075 ms
 7  80.67.4.134 [AS42708]  83.246 ms 80.67.4.132 [AS42708]  84.578 ms 80.67.4.134 [AS42708]  84.524 ms
 8  72.14.216.118 [AS15169]  85.291 ms  85.403 ms  85.250 ms
 9  * * *
10  8.8.8.8 [AS15169]  84.948 ms  84.926 ms  85.328 ms
```

Трафик идет через: AS42708, AS15169

***
## Повторите задание 5 в утилите `mtr`. На каком участке наибольшая задержка - delay?
###
```bash
dmitriy@dmitriy-lin:~$ mtr -rnc 10 8.8.8.8
Start: 2023-02-08T17:44:39+0700
HOST: dmitriy-lin                 Loss%   Snt   Last   Avg  Best  Wrst StDev
  1.|-- 192.168.100.1              0.0%    10    0.4   0.5   0.4   0.6   0.1
  2.|-- 10.100.10.1                0.0%    10   51.3  51.3  51.1  51.6   0.1
  3.|-- 10.77.77.10                0.0%    10   90.4  95.5  90.4 112.4   6.7
  4.|-- 46.246.96.1                0.0%    10  113.0  99.2  91.2 115.5   8.8
  5.|-- 80.67.3.188                0.0%    10  100.4  99.1  91.5 108.3   5.6
  6.|-- 80.67.4.156                0.0%    10   95.6 104.4  92.3 159.9  21.3
  7.|-- 80.67.4.134                0.0%    10   93.9 109.2  91.8 185.7  28.2
  8.|-- 72.14.216.118              0.0%    10   92.1  95.1  91.8 106.9   4.8
  9.|-- 142.250.213.135            0.0%    10   92.0  99.7  91.2 114.9   8.3
 10.|-- 209.85.241.225             0.0%    10   92.0 102.8  92.0 134.2  12.4
 11.|-- 8.8.8.8                    0.0%    10   91.9 100.3  91.8 128.0  12.0
```
Наибольшая задержка:
	* средняя: 109.2 (7)
	* максимальная: 185.7 (7)

***
## Какие DNS сервера отвечают за доменное имя dns.google? Какие A записи? Воспользуйтесь утилитой `dig`
###
```bash
dmitriy@dmitriy-lin:~$ dig +trace dns.google | grep dns
; <<>> DiG 9.16.1-Ubuntu <<>> +trace dns.google
dns.google.		10800	IN	NS	ns2.zdns.google.
dns.google.		10800	IN	NS	ns1.zdns.google.
dns.google.		10800	IN	NS	ns3.zdns.google.
dns.google.		10800	IN	NS	ns4.zdns.google.
dns.google.		3600	IN	DS	56044 8 2 1B0A7E90AA6B1AC65AA5B573EFC44ABF6CB2559444251B997103D2E4 0C351B08
dns.google.		3600	IN	RRSIG	DS 8 2 3600 20230228201213 20230206201213 48465 google. KoIbvYINfVsirxgIFQwKMHP+wbII4SPW5sUXVeTqz/OQCVPnDA44DNJs EnEZTjER3csWeyOcs/a4+4gmnhsvT66TnrkecbZqFxtimNtIphloErZz eQHS4wEPI357sOD1jFgwtA60sSREpEcotsNmCkqZhrCzSER+Zzq/PHj0 FxM=
dns.google.		900	IN	A	8.8.8.8
dns.google.		900	IN	A	8.8.4.4
dns.google.		900	IN	RRSIG	A 8 2 900 20230227125252 20230205125252 35583 dns.google. aE4ikQYUa75gQlxoG8Vl0LhWSKd6t4D8X7h/aR3V4tKIIYdD83MDM0ew p3swFlhNihI5YxqT3f6IdzpgaOsO/LCz4Os7quvBoR6vnav4aPJL/MhP KRU6J2kjs6CNWNb+ac+t/LU+bTiRuLQbWHk6vVxAoL3NPqURXMKgQphn aQg=
;; Received 241 bytes from 216.239.34.114#53(ns2.zdns.google) in 104 ms
```
A записи: 8.8.8.8 и 8.8.4.4
NS сервера: ns1.zdns.google.; ns2.zdns.google.; ns3.zdns.google.; ns4.zdns.google.

***
## Проверьте PTR записи для IP адресов из задания 7. Какое доменное имя привязано к IP? Воспользуйтесь утилитой `dig`
###
```bash
dmitriy@dmitriy-lin:~$ dig -x 8.8.4.4 +noall +answer
4.4.8.8.in-addr.arpa.	19588	IN	PTR	dns.google.
dmitriy@dmitriy-lin:~$ dig -x 8.8.8.8 +noall +answer
8.8.8.8.in-addr.arpa.	6643	IN	PTR	dns.google.
dmitriy@dmitriy-lin:~$
```
Оба имеют PTR dns.google.