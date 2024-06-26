---
title: Wybór systemu.
author: brat
date: 2024-04-03 00:00:00 +0000
categories: [Homelab]
tags: [software, docker, os, home assistant, proxmox, truenas, backup]
render_with_liquid: false
---


## Jakie rozwiązanie warto zastosować?

Można powiedzieć, że wróciłem do punktu wyjścia. Kolejny raz rozważam, jaki system operacyjny lub jaka kombinacja systemów byłby najlepsza w mojej sytuacji. W swoim homelabie mam jeden skromny serwer wirtualny ([Home Assistant](https://www.home-assistant.io/)) plus około 30 kontenerów. Większość z nich jest używana na bieżąco, typu [Authentik](https://goauthentik.io/), [Vaultwarden](https://github.com/dani-garcia/vaultwarden), bazy danych; część włączam w razie potrzeby, np: [Code-server](https://github.com/coder/code-server), server minecraft z użyciem [Pterodactyla](https://pterodactyl.io/).

Do rozdysponowania mam 3 serwery, więc możliwości jest wiele. Która z nich jest najlepsza? Nie wiem, dlatego też można poniższe wypociny potraktować jako notatnik, w którym staram się wytłumaczyć mój sposób myślenia. Dla przypomnienia - poniżej to, z czym pracujemy:

|          |      CPU      |  RAM  |    Dyski    |
|----------|:-------------:|:-----:|:-----------:|
| Server 1 |    i3-6100    |  8 GB | 0,25 + 1 TB |
| Server 2 |    i3-8100    | 16 GB |   2x10 TB   |
| Server 3 | Celeron E3400 |  2 GB |    0,5 TB   |


Do tej pory rozważałem:
### Wariant z Proxmoxem. 
Na serwer główny (w tej dumnej roli Server 1) wrzucamy [Proxmoxa](https://www.proxmox.com/en/). Na nim mamy odpaloną naszą maszynę wirtualną + usługi w kontenerach LXC (a może na kolejnej wirtualce, pod kontrolą dockera?). Na serwerze 2 instalujemy Proxmox Backup Server. Natomiast Server 3 - do ustalenia, czy będzie to coś z rodziny Proxmox, czy czyste Ubuntu, udostępniające swoje dyski.
#### Zalety
* Dostępna duża ilość poradników, zarówno w formie video jak i opisowej
* Rozwiązania problemów łatwe do znalezienia w necie
* Duża elastyczność, zmiany robi się w VM'kach/kontenerach a nie w hypervisorze

#### Wady
* Nie znam dobrze proxmoxa, nawet proste rzeczy mogą zająć dużo czasu
* Instalowałbym pierwszy raz, więc może się okazać po czasie, że muszę coś instalować od nowa, bo np. coś można ustawić tylko w momencie instalacji

### Wariant z migracją Home Assistanta do kontenera. 
W takim wypadku można wszystko na Serwerze 1 postawić w dokerze pod kontrolą Ubuntu, dla "wygody" uruchomić portainer (choć w sumie nie jest to niezbędne, z moją znajomością dockera i jego CLI). Serwer 2 i 3 zostaną z Truenas i Ubuntu. 
#### Zalety
* Dostępna duża ilość poradników, zarówno w formie video jak i opisowej
* Rozwiązania problemów łatwe do znalezienia w necie
* Duża elastyczność, zmiany robi się w kontenerach a nie w systemie operacyjnym

#### Wady
* Czytałem, że łączenie różnych usług/dodatków w Home Assistancie może być kłopotliwe
* Brak systemowego rozwiązania do backupu


### Wariant mieszany.
W związku, że największe doświadczenie mam z systemem UnRaid i spełnia on moje wymagania, to zostaje on na Serwerze 1. Na Serwer 2 wrzucam natomiast Truenas Scale. Serwer 3 - zostaje bez zmian.

#### Zalety
* Mam największe doświadczenie z Unraid
* Będę miał bezpieczeństwo zapewnione przez softraid w Unraid jak i ZFS w Truenasie
* Użycie dwóch bardzo popularnych systemów zapewni wsparcie w razie problemów
* Licencje na Unraid już posiadam

#### Wady
* Brak systemowego rozwiązania backupów
* Brak centralnego zarządzania usługam



### Warianty które brałem pod uwagę, ale odrzuciłem.
W każdym z nich Serwer 3 zostaje na Ubuntu, więc nie będę go wymieniał za każdym razem.
* Wrzucić Home Assistanta na Server 1 jako główny system i resztę usług odpalić z kontenerów. Serwer 2 - Truenas. 
* na Serwer 2 wrzucić OpenMediaVault i zobaczyć co to za wynalazek.
* Dwa serwery Truenas.
* Zamiast jednego serwera - zainwestować w backup w chmurze, np Back Blaze Personal lub któryś z dostępnych S3.
* Użycie Xpenology

Niedługo będę musiał podjąć decyzję. Jako, że jestem leniwy, nie będzie mi się chciało robić instalacji kilka razy, więc wybór jednego rozwiązania wcale nie jest taki prosty.

