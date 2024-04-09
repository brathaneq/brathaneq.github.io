---
title: Pierwsze testy.
author: brat
date: 2024-04-09 10:00:00 +0000
categories: [Homelab]
tags: [software, hardware, truenas, hp, server, ventoy]
render_with_liquid: false
---


## Pierwsze testy na nowym sprzęcie

W końcu udało mi się wygospodarować trochę czasu, żeby zająć się moim małym projektem. Przy okazji generalnych porządków (przedmuchanie, wymiana pasty, wyczyszczenie obudowy) do Elitedeska wrzuciłem dyski (dość dziwna mieszanka, bo 1TB (5900 RPM), 3 TB (7200 RPM) i 250GB SSD). Przy pierwszym uruchomieniu chciałem sprawdzić, czy w BIOS jest odpowiednio ustawiony (m.in włączona wirtualizacja, włączone stany oszczędzania energii, kolejność bootowania, zachowanie przy zanikach napięcia). 

### Początek przygód 

Wprawdzie komputer ma dwa wyjścia wideo, to są to Display Porty, a ja jak na złość nie mam ani adaptera do HDMI, ani odpowiedniego kabla. Nie chciałem generować niepotrzebnych kosztów, więc wrzuciłem tymczasowo (nie sądziłem, że AŻ TAK krótko, więcej o tym ciut niżej) starą kartę graficzną i podłączyłem wszystko do telewizora. Jako, że moje serwery działają w szafie na poddaszu, to nie miałem do tej pory potrzeby inwestowania w monitor. Raz na dwa lata mogę wykorzystać telewizor do konfiguracji. Po pierwszym włączeniu, zanim znalazłem odpowiednią kombinację klawiszy to odpalił się Windows! No tak, na starym dysku ktoś musiał wrzucić świeży system. Nie wnikając, po resecie w końcu znalazłem odpowiednią kombinację i mogłem ustawić co chciałem.

### Żegnaj Windowsie, witaj Truenasie

Jakiś czas temu przygotowałem sobie z pomocą [Ventoy](https://www.ventoy.net/) pendrive USB z obrazami ISO różnych dystrybucji które brałem pod uwagę. Do końca nie byłem pewien co zainstaluję, jednak jak pojawiła się opcja wyboru obrazu - wybrałem Truenasa w wersji scale. Instalacja jest bezproblemowa, nie będę jej tutaj opisywał. Po wyborze dysku do instalacji i wyborze hasła wszystko leci automatycznie. 

### Pierwsze potknięcie
Po skończeniu instalacji i reboocie, system ochoczo zaczął się odpalać. Jednak, ku memu zdziwieniu, po krótkiej chwili uruchamianie się zatrzymało z komunikatem "VT-d active for GFX access".  
 ![zdjęcie poglądowe](https://preview.redd.it/first-system-boot-stuck-on-vt-d-active-for-gfx-access-v0-9uup2wd77j8a1.jpg?width=1080&crop=smart&auto=webp&s=34780bca3f5c78a0e414e457647d654f1291b193)
_zdjęcie poglądowe z Reddita_

Jak tylko zacząłem wpisywać w Google _VT-d act_ reszta kodu błędu się podpowiedziała, więc była bardzo duża szansa, sytuacja w której się znalazłem zdarza się na tyle często, że bez problemu powinienem znaleźć rozwiązanie. I faktycznie tak było. Pierwszy link w [Reddit](https://www.reddit.com/r/truenas/comments/zwnev5/first_system_boot_stuck_on_vtd_active_for_gfx/) daje od razu rozwiązanie - usunąć kartę graficzną...
W moim przypadku nie był to problem, bo system docelowo i TAK miał działać bez monitora, więc wyjąłem GPU. Kiedy komputer się uruchamiał sprawdziłem w ustawieniach rutera jaki adres IP został przypisany do Truenas (ku memu zdziwieniu został rozpoznany jako Mac Pro :smirk: ).

### Pierwsze logowanie

Po wpisaniu wyżej wspomnianego adresu IP do przeglądarki oczom mym ukazał się ekran logowania.
![Truenas login](/assets/2024-04-09/truenas-login.jpg).

Nigdy wcześniej nie zapoznawałem się z poradnikami z Truenasa, więc trochę metodą prób i błędów zrobiłem swój pierwszy zasób z dysków i zapoznałem się z interfacem.
Aktualnie sprawdzam pobór prądu, przy zerowym obciążeniu. Za kilka dni wrzucę trochę aplikacji i zobaczę jak system się zachowuje pod minimalnym obciążeniem. 

Zapraszam na więcej już niedługo :)