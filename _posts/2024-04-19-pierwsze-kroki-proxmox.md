---
title: Pierwsze kroki w Proxmox.
author: brat
date: 2024-04-19 00:00:00 +0000
categories: [Homelab]
tags: [software, proxmox, github]
render_with_liquid: false
---
## Pierwsze kroki z Proxmox.

Cześć!
Mamy już działający serwer z Proxmoxem, poniżej kilka pierwszych kroków które warto wykonać na samym początku poznawania tego systemu. Proponuję zapoznać się najpierw z całym wpisem, a dopiero potem przejść do jego ewentualnej realizacji.

### Zabezpieczanie logowania
Dobrą praktyką jest jak najszybsze zabezpieczenie konta używanego do logowania się do serwera.
Klikamy `Datacenter --> Permissions` i po rozwinięciu listy wybieramy Two Factor.
![Persmissions](/assets/img/2024-04-19/2fa-1.jpg)
_wybór okna TOTP_
Następnie wybieramy _TOTP_ z listy rozwijanej, która pojawia się po naciśnięciu `Add`.
Pojawia się nam coś takiego:
![TOTP Config](/assets/img/2024-04-19/2fa-2.jpg)
_konfiguracja TOTP_
W polu `Description` wpisujemy opis, np informację jakiej aplikacji będziemy używać do generowania kodu. Możliwości jest wiele, [Authy](https://authy.com/), [Google Authenticator](https://play.google.com/store/apps/details?id=com.google.android.apps.authenticator2), [MS Authenticator](https://www.microsoft.com/pl-pl/security/mobile-authenticator-app), ja natomiast użyję [Bitwardena](https://bitwarden.com/).
Używając aplikacji mobilnej skanujemuy kod QR i dodajemy Proxmoxa do listy obsługiwanych kont/aplikacji. Alternatywnie - można skopiować zawartość pola `Secret` i wkleić go w odpowiednie pole w aplikacji obsługującej TOTP. Rezultat będzie identyczny - aplikacja będzie generowała co 30 sekund sześciocyfrowy kod. W polu `Verify Code` wpisujemy nasz jednorazowy kod. Po kliknięciu 'Add' mamy sukces!
![TOTP Sukces](/assets/img/2024-04-19/2fa-3.jpg)

Od tej pory każde logowanie użytkownika będzie wymagało dodatkowego potwierdzenia!
Skoro już jesteśmy w konfiguracji dostępu - wygenerujmy kody tzw. recovery, których można użyć w przypadku problemów z wygenerowaniem kodu TOTP (np. utrata telefonu). Klikamy ponownie w `Add` i wybieramy _Recovery Keys_. Po wybraniu użytkownika klikamy `Add`. Pojawi się lista kodów podobna jak na obrazie poniżej:
![recovery keys](https://pve.brathaneq.eu.org/pve-docs/images/screenshot/pve-gui-tfa-add-recovery-keys.png)
_przykład z oficjalnej dokumentacji_
Kody drukujemy/zapisujemy i trzymamy w bezpiecznym miejscu.


### Usunięcie repozytoriów Enterprise 
Klikamy na nasz node (w naszym przypadku `pve`), potem Updates i Repositories.  
Pojawi się nam lista używanych repozytoriów:
![spis](https://user-images.githubusercontent.com/63487881/203458586-48eb767c-ac27-4362-978c-092841238959.png)
_Źródło: https://github.com/CarmineCodes/Proxmox-No-Subscription-No-Problem_

Wybieramy repozytorium z dolnej listy (Enterprise) i klikamy na `Disable`.
Następnie klikamy `Add` i z listy rozwijanej wybieramy No Subscription.

![dodawanie repozytorium](https://user-images.githubusercontent.com/63487881/203459229-88135996-491b-4ccc-97a2-aa10d2becb5c.png)
_Źródło: https://github.com/CarmineCodes/Proxmox-No-Subscription-No-Problem_

Teraz przy aktualizacji aplikacji i systemu nie powinny nam się pojawiać błędy/ostrzeżenia.

### Aktualizacja systemu z wersji 7 do wersji 8
Wcześniejszy krok zaczął nas przygotowywać do aktualizacji systemu, ponieważ będzie trzeba pościągać sporą ilość danoch z nowych repozytoriów. Na szczęście cała procedura jest bardzo dokładnie opisana [tutaj](https://pve.proxmox.com/wiki/Upgrade_from_7_to_8), można znaleźć też na Youtube [filmy](https://www.youtube.com/watch?v=i5cmx-mcUVA), gdzie krok po kroku pokazana jest ta aktualizacja.

> Zanim zaczniemy cokolwiek robić na serwerze - musimy się z nim połączyć przez SSH, lub zalogować się lokalnie z konsoli (klawiatura + monitor). Nie można stosować wbudowanego w GUI terminala, ponieważ połączenie zostanie zerwane w trakcie aktualizacji. A tego byśmy nie chcieli.
{: .prompt-warning }

#### Sprawdzamy gotowość do aktualizacji
Po zalogowaniu jako root - wpisujemy 
```bash
pve7to8
```
Powinna nam się po chwili pojawić lista wyników testów, czy system jest gotowy do aktualizacji. W moim przypadku nie było ani błędów, ani ostrzeżeń, więc mogłem przejść do nastęnego kroku. 
>Przypominam, że w moim przypadku nie tworzyłem jeszcze żadnych kontenerów/maszyn wirtualnych, więc proces przygotowania do aktualizacji jest uproszczony.
{: .prompt-info }

#### Aktualizacja zasobów
Kolejnym krokiem jest sprawdzenie, czy pakiety które mamy w systemie są aktualne. Można to zweryfikować komendami:
```bash
apt update
apt dist-upgrade
pveversion
```
Po wydaniu ostatniej komendy powinniśmy otrzymać wersję `7.4-15` lub wyższą.

#### Zmiana repozytoriów
Jeżeli wszystko do tej pory przebiegało poprawnie, kolejną rzeczą jest zmiana repozytoriów na nowe:
```bash
sed -i 's/bullseye/bookworm/g' /etc/apt/sources.list
```
w filmiku na Youtube, który jest wstawiony wyżej, autor weryfikuje poprawność zmiany sprawdzając listę repozytoriów poleceniem `cat`
```bash
cat /etc/apt/sources.list.d/pve-enterprise.list
cat /etc/apt/sources.list
```
>Zwracam uwagę, że pve-enterprise.list powinno być poprzedzone znakiem # bo zostało ono wyłączone we wcześniejszych krokach.
{: .prompt-info }

#### Dość przygotowań, zmieńmy w końcu 7 na 8!
>Przypominam, że poniższych komend NIE możemy wykonywać na terminalu wbudowanym w GUI proxmoxa!
{: .prompt-warning }
Wszystko jest gotowe, więc wydajemy po kolei komendy:
```bash
apt update
apt dist-upgrade
```
Trochę to potrwa, od kilku do kilkunastu minut. W trakcie zostaniemy kilka razy spytani, jak chcemy postępować z niektórymi pakietami. Ja osobiście, jako, że to maszyna testowa, zgadzałem się na aktualizację wszystkich pakietów. W waszym przypadku może być inaczej, [tutaj](https://pve.proxmox.com/wiki/Upgrade_from_7_to_8#Upgrade_the_system_to_Debian_Bookworm_and_Proxmox_VE_8.0) jest to dokładniej opisane w oficjalnej dokumentacji.

Jak proces instalacji dobiegnie końca - możemy zrestartować komputer wpisując komendę `reboot`, lub klikając w Reboot w prawym górnym rogu ekranu, jak jesteśmy zalogowani na GUI Proxmoxa.
Po restarcie i odświeżeniu okna w przeglądarce, której używamy do logowania się do Proxmoxa, powinniśmy zobaczyć nową, zaktualizowaną wersję, w moim przypadku jest to 8.1.10.

### Wyłączenie komunikatu o braku subskrypcji
Jeżeli nie mamy wykupionej [płatnej subskrypcji](https://www.proxmox.com/en/proxmox-virtual-environment/pricing) dającej dostęp do repozytoriów Enterprise - przy każdym logowaniu będzie nam się pojawiało okienko z komunikatem:
![subscription alert](/assets/img/2024-04-19/subscription.jpg)
Niby to tylko jedno kliknięcie, ale po jakimś czasie zaczyna to drażnic. Dlatego też - szybko postaramy się to wyłączyć.
Klikamy na nasz node (w naszym przypadku `pve`) i wybieramy `Shell`.
Tam wpisujemy jedną komendę:

```bash
sed -Ezi.bak "s/(Ext.Msg.show\(\{\s+title: gettext\('No valid sub)/void\(\{ \/\/\1/g" /usr/share/javascript/proxmox-widget-toolkit/proxmoxlib.js && systemctl restart pveproxy.service
```
W zależności od konfiguracji przeglądarki powinno się wyczyścić cache, zamknąć zakładkę, lub uruchomić ponownie całą przeglądarkę. 

## Pomoc z Discorda
Szukając informacji dot. konfiguracji dobrzy ludzie z Discorda (dzięki @michal_32) podpowiedzieli mi użycie skryptów ze Strony [Helper Scripts](https://tteck.github.io/Proxmox/). Jest to zbiór skryptów na każdą okazję, warto się z nimi zapoznać, mogą usprawnić naprawdę wiele czynności. Jednym ze skryptów jest Post Install w kategorii `Proxmox VE Tools`. Robi w sumie wszystko to co opisałem wyżej, tylko automagicznie i z minimalnym nakładem pracy użytkownika.

## Co dalej?
Wiele rzeczy przed nami, w ciągu kilku następnych dni powinnny się pojawić kolejne wskazówki co robimy na początku pracy na Proxmoxie. Włączanie grup IOMMU, zakładanie użytkowników (w nieoczywisty sposób), uruchamianie pierwszych kontenerów LXC i wiele wiecej. 
Zapraszam niedługo!
