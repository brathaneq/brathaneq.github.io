---
title: Problem z Authentikiem.
author: brat
date: 2024-03-29 11:00:00 +0000
categories: [Blog, Docker]
tags: [software, docker, authentik, recovery, homelab, blog, security]
render_with_liquid: false
---


## Problem z Authentikiem

Witam wszystkich bardzo serdecznie i zapraszam do zapoznania się z moją dzisiejszą przygodą.  
Regularnie loguje się na moje usługi, żeby sprawdzić, czy nie ma np. jakichś aktualizacji, które można by zastosować, czy nie pojawiło się coś o czym powinienem wiedzieć. Ku mojemu wielkiemu zaskoczeniu, napotkałem błąd przy logowaniu. Wszystkie moje usługi, które są wystawione na "świat" są zabezpieczone [authentikiem](https://goauthentik.io/).  
Po wpisaniu poprawnie loginu, hasła oraz przepisaniu kodu TOTP moim oczom pokazał się następujący komunikat:

![Invalid token](/brathaneq.github.io/assets/img/2024-03-29/auth1.jpg) 

Przetestowałem klienta z telefonu, jak i kod wygenerowany z aplikacji w komputerze. 
Wiedząc na jakiej zasadzie działają takie kody, pierwsza rzecz jaką zrobiłem, to sprawdzenie, czy czas w authentiku jest ustawiony poprawnie. Jako, że działa on w kontenerze wymagało to prostej komendy:

```bash
brat@Node: docker exec authentik date && date  
Fri Mar 28 18:33:16 CET 2024
Fri Mar 28 18:33:16 CET 2024
```

"Niestety" okazało się, że czas który podaje kontener pokrywa się z czasem w komputerze, czasem w komórce, czasem w Internecie. Więc szukam dalej. Przecież mam regularnie robione kopie bezpieczeństwa. Ty też, prawda ? ;) I znowu porażka, mimo przywróceniu kopii bezpieczeństwa authentika sprzed tygodnia, ciągle wpisywany token nie był rozpoznany. Z perspektywy czasu, myślę, że mógłbym jeszcze przywrócić bazę danych PostgreSQL, ale zanim do tego dotarło, to znalazłem inny sposób.  
Na oficjalnej stronie znalazłem sposób na reset hasła, dokładnie opisany [tutaj](https://docs.goauthentik.io/docs/troubleshooting/login).

Mój authentik, jak już wspominałem, jest w kontenerze, więc musiałem wziąć komendę CLI:
```bash
docker exec authentik ak create_recovery_key 10 <twój_login>
```
Po użyciu tej komendy po chwili na ekranie pokaże się coś takiego:
```
Store this link safely, as it will allow anyone to access authentik as <twój_login>.
/recovery/use-token/mOeHaibJyExLHMByPCChKepZ7ICkp3dYtbZdaRlyjS4mHxMMrTDRtkAopmaL/
```
który da dostęp jako użytkownik `twój_login`. Należy to dokleić na koniec domeny pod którą jest udostępniony authentik, żeby powstało coś takiego:
```
https://authentik.company/recovery/use-token/mOeHaibJyExLHMByPCChKepZ7ICkp3dYtbZdaRlyjS4mHxMMrTDRtkAopmaL/
```

Po wklejeniu powyższego do pola adres w przeglądarce - przeniesieni będziemy do panelu użytkownika **bez podawania loginu i hasła!**  
>Uwaga, Token jest ważny przez 10 lat, więc po jego użyciu lepiej go skasować. o Tym będzie trochę niżej.
{: .prompt-warning }

Stamtąd można wejść w ustawienia konta (kółko zębate w prawym górnym rogu ekranu) i w zakładce `MFA Devices` znaleźć starą pozycję TOTP Authenticator, zaznaczyć ją i skasować.
![kasowanie TOTP](/brathaneq.github.io/assets/img/2024-03-29/auth2.jpg)

Następnie klikamy na niebieski guzik "Enroll" i wybieramy z listy rozwijanej jaki rodzaj uwierzytelniania dwuskładnikowego chcemy dodać. Ja wybrałem najpierw **TOTP**, a potem dla pewności, dodałem również **WebAuth** i **Static Token**, żeby w przyszłości prościej sobie poradzić z takimi przygodami.  
Po dodaniu 2FA do konta proponuję udać się do zakładki `Tokens and App passwords` i tam usunąć token dostępowy, który przed chwilą wygenerowaliśmy w linii poleceń.
I w ten sposób wracamy do normalności - znowu wszystko powinno działać.  

Do następnej przygody!
