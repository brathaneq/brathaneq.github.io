---
title: Zmiany, zmiany.
author: brat
date: 2024-04-18 00:00:00 +0000
categories: [Homelab]
tags: [software, truenas, proxmox, nvidia, gt710, server, ventoy]
render_with_liquid: false
---
## Zmiany, zmiany.

Cześć!
Minęło kilka dni, więc warto napisać, ku potomności, co dzieje się ciekawego. 

### Truenas

Niestety system nie został ze mną na długo. Nawet nie dlatego, że coś było z nim nie tak. Po zainstalowaniu i ogarnięciu podstawowych ustawień (spięcie dysków w Pool, udostępnienie zasobów w sieci, zainstalowanie kilku aplikacji) system działał stabilnie. Kiedy był w idle – dyski przestawały się kręcić, co pozwalało zaoszczędzić trochę energii. Średnio komputer bez obciążenia pobierał jakieś **16W**. Myślę, że z użyciem samych dysków SSD dałoby się zejść jeszcze niżej, ale jest to już wynik lepszy niż mojego drugiego komputera (okolice 35W-40W). Niemniej – po tylu latach z CLI i Unraidem interface jakoś mnie nie pociągał. Truenas robiłby za dobrego konia pociągowego, doceniam ZFS, snapshoty, mechanizmy backupu wbudowane w system. Myślę, że to nie jest ostatni raz kiedy widzicie ten system na tym blogu. Ale czas było sprawdzić następnych kandydatów z listy!

### Żegnaj Truenasie, witaj Proxmoxie
Stało się. Zarzekałem się jeszcze niedawno, że nie będę korzystał z tego wynalazku. Zielone wykresy zużycia RAMu i CPU działały na mnie odstraszająco 😉 Ale cóż, trzeba było spojrzeć prawdzie w oczy, jest to rozwiązanie bardzo rozbudowane, żeby nie rzec kompletne, popularne, dopracowane, na stabilnym fundamencie (Debian) i warto było to sprawdzić osobiście.

#### Instalacja
W Internecie jest masa poradników, zaczynając od [oficjalnego](https://pve.proxmox.com/pve-docs/chapter-pve-installation.html#installation_installer), przez instrukcje na blogach/reddit, kończąc na [filmach na YT](https://www.youtube.com/watch?v=sZcOlW-DwrU). Pomyślałem, że co może być w tym trudnego, instalacja jak wiele innych. Na USB z Ventoy miałem już plik ISO, więc odpalenie instalatora to była pestka. Przy okazji podłączenia komputera do ekranu poustawiałem kilka rzeczy w BIOSie o których zapomniałem/źle ustawiłem wcześniej. Wybrałem tryb graficzny i .. pierwszy problem. Po wyborze standardowej instalacji system zatrzymuje się na ładowaniu sterowników. ![zdjęcie poglądowe](https://preview.redd.it/proxmox-ve-8-1-install-freezes-during-loading-drivers-v0-0y61xjy4axac1.png?width=1026&format=png&auto=webp&s=5c020fe5dc508cebedfedddb0a4634374f2bdfed)
_zdjęcie poglądowe z Reddita_

OK. Myślałem, że to nie jest duży problem, dlatego też odpaliłem tryb tekstowy (który okazał się zaskakująco graficzny) z takim samym rezultatem. Wiedząc, że bez porady Wujka Google się nie obejdzie – zacząłem szukać rozwiązania. Szybko udało mi się zawęzić krąg podejrzanych do GPU lub wersji proxmoxa. Niestety nie miałem możliwości zmiany GPU (przypomnę, GT710) więc spróbowałem zmienić wersję z 8.1 na 7.4. 

#### Instalacja, podejście nr 2
Ku memu niezadowoleniu, zmiana wersji nie rozwiązała problemu. System dalej nie chciał uruchomić instalatora. Nie miało znaczenia czy włączałem tryb graficzny, czy tryb tekstowy.
Wujek Google podpowiedział, żeby dodać parametr `nomodeset` przy bootowaniu, szerzej problem opisany m.in. [tutaj](https://forum.proxmox.com/threads/proxmox-8-installer-freezes-at-boot.129341/). Po zalecanej modyfikacji – w końcu rozpoczął się proces instalacji. Kilka kliknięć później komputer restartował się już z gotowym systemem. 
Oczywiście, byłoby nudno jakby zadziałało od razu…
Komputer zatrzymywał się przy komunikacie „Loading initial ramdisk …” ![ekran błędu](https://img2023.cnblogs.com/blog/27422/202308/27422-20230830100726995-422618091.png)
_zdjęcie poglądowe_

Mamrocąc pod nosem „stabilny, dopracowany” pomyślałem, że może to znowu problem z grafiką. Wyciągnąłem kartę z obudowy i po chwili od odpalenia – serwer pojawił się na liście podłączonych urządzeń na ruterze. Sprawdziłem z laptopa, czy mogę się zalogować na Proxmoxa, oczom moim ukazało się tak wyczekiwane okienko:

![ekran logowania](/assets/img/2024-04-18/login.jpg)_długo wyczekiwany ekran logowania_

#### Sukces!
Złożyłem Elitedeska, włożyłem go z powrotem do szafy rack i zacząłem układać w myślach listę „co dalej”. I właśnie ta lista to będą moje kolejne wpisy, do których serdecznie zapraszam! 😊