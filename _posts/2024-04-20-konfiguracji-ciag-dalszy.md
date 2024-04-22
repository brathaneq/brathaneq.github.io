---
title: Dalsza konfiguracja Proxmoxa
author: brat
date: 2024-04-20 00:00:00 +0000
categories: [Homelab]
tags: [software, proxmox, authentik, ubuntu]
render_with_liquid: false
---
## Kolejne kroki

Cześć!
Dziś kontynuujemy konfigurację serwera, żeby w KOŃCU, zacząć uruchamiać z niego jakieś usługi.

### Ciemny motyw
Zacznijmy od zmiany, która ochroni nasze oczy ;) Zmiana wyglądu systemu z jasnego na ciemny jest bardzo prosta:
1. W prawym gónym rogu ekranu klikamy nasz przycisk z naszym loginem:
![zmiana wyglądu](/assets/img/2024-04-20/theme.jpg)
2. Wybieramy `Color Theme`
2. Z menu rozwijanego wybieramy Proxmox Dark i klikamy Apply
   
Zrobione! 

### Konfiguracja nowego użytkownika (with a twist!)
Na początku chciałem opisać sposób skonfigurowania domeny (na bazie cloudflare) oraz późniejsze dopisanie nasezgo serwera do Traefika (Reverse proxy). Niestety, o ile konfiguracja domeny jest względnie uniwersalna, to konfiguracja reverse proxy już jest ciut bardziej zależna od aktualnej konfiguracji, więc musze założyć, że jak ktoś ma już traefik skonfigurowany to wie jak dodać do niego kolejną usługę. Ale skoro jesteśmy już przy wystawieniu naszego serwera na świat, to należy pamiętać, żeby robić to w sposób jak najbardziej bezpieczny. Do naszych celów użyjemy [Authentika](https://goauthentik.io/). Mamy do wyboru 2 opcje:
* użyć Authentika jako bramy dostępowej do Proxmoxa.
* użyć wbudowanej w Proxmoxa usługi OpenID

#### Authentik jako brama
Sposób na użycie Authentika jest kilka, jednym z nich jest konieczność uwierzytelnienia się, żeby w ogóle zobaczyć ekran logowania. Można to zrobić też na kilka sposób:
* proxy wbudowane w Authentik
* ustawienie forward 
![proxy](/assets/img/2024-04-20/provider-proxy.jpg)

Jest to kwestia odpowiedniej konfiguracji Providera, i absolutnie nie wymaga żadnych zmian w konfiguracji Proxmoxa. Wadą takiego rozwiazania jest to, że musimy się autoryzować dwa razy. Raz w Authentiku, i drugi raz w Proxmoxie. 
Dlatego też, użyjemy drugiej opcji, czyli OpenID.

#### Konfiguracja OpenID z użyciem Authentika
Konfigurując to pierwszy raz - opierałem się na [dokumentacji](https://docs.goauthentik.io/integrations/services/proxmox-ve/).

##### Krok 1 - przygotowanie Providera
Logujemy się na konto administratora Authentika i wchodzimy w panel administracyjny.
Tam - wybieramy `Aplications -> Providers` i klikamy `Create`.
Z listy wybieramy 'OAuth2/OpenID Provider' i klikamy 'Next'.
Uzupełniamy nazwę oraz flow do poświedczenia i autoryzacji (jak już masz skonfigurowanego Authentika, to pewnie masz to już gotowe. Jeżeli nie, to na Youtube są świetne [tutoriale](https://www.youtube.com/playlist?list=PLH73rprBo7vSkDq-hAuXOoXx2es-1ExOP) do Authentika autorstwa Cooptonian). 
Spisujemy Client ID i Secret Secret, przyda nam się później (spokojnie, można to też podejrzeć później). Uzupełniamy polę 'Redirect URIs/Origins (RegEx)' naszym adresem internetowym Proxmoxa. W moim przypadku nie musiałem podawać portu, odpowiednie przekierowanie robi już traefik. Zgodnie z wytycznymi dokumentacjami nie dajemy '/' na końcu adresu.
> Dodatkowo w zakładce `Advanced protocol settings` w polu `Subject mode` wybieramy opcję `Based on User's username`. Pozwoli to na czytelniejsze opisywanie użytkowników w Proxmoxie. 
{: .prompt-info }
Po zatwierdzeniu Providera - klikamy w jego nazwę i sprawdzamy co znajduje się w polu `OpenID Configuration Issuer`. Przyda nam się to później. 

##### Krok 2 - przygotowanie Aplikacji
Następnie wybieramy `Aplications -> Aplications` i klikamy `Create`.
Uzupełniamy pola `Name` i `Slug`, z listy `Providers` wybieramy Providera którego zrobiliśmy we wcześniejszym kroku. Po rozwinięciu opcji `UI Settings` uzupełniamy pole `Launch URL` pełnym adresem internetowym Proxmoxa (tutaj już z / na końcu).

##### Krok 3 - konfigurujemy Proxmoxa
Przechodzimy do `Datacenter -> Permissions -> Realms` i po kliknięciu w `Add` wybieramy OpenID Connect Server.
Wypełniamy jak niżej, z zastrzeżeniem, że `authentik.company` zmieniamy na nasz adres internetowy Authentika.
![Proxmox-realm](https://docs.goauthentik.io/assets/images/proxmox-source-7a3928943e502407ff132fb872f9be48.png)
_Źródło: dokumentacja Authentik_
To powinna być ta sama wartość jaką sprawdzaliśmy w Kroku 1.

Gotowe! Teraz przy logowaniu się do naszego serwera Proxmox, można wybrać nową opcję - `authentik`.
![realm-choice](/assets/img/2024-04-20/realm-choice.jpg).
Po jej wybraniu będziemy mieli możliwość przekierowania procesu logowania na zewnętrzny serwer.
Przechodzimy procedurę uwierzytelniania w Authentik i sukces!
![brat@authentik](/assets/img/2024-04-20/brat-auth.jpg)

##### Bonus: Krok 4 - nadanie uprawnień
Logujemy się na konto administratora (zmieniając `Realm` na `Linux PAM standard authentication)` i klikamy w `Permissions` z menu po lewej stronie. Tam wybieramy `Add` i `User Permission` i wypełniamy następująco:
* Path - wpisujemy `/`
* User - wybieramy nasze konto z authentika
* Role - ja na czas testów wybrałem `Administrator`


### Włączenie grup IOMMU
Na początku użytkowania systemu - wszystkie zasoby są przypisane do hypervisora. Czasami jednak chcemy jakiś sprzęt "przekazać" do maszyny wirtualnej. Czy to dysk twardy, kontroler dysków, karta sieciowa, karta graficzna, wszystko zależy od tego co potrzebujemy użyć w sposób bezpośredni z poziomu maszyny wirtualnej.
Najprostszym przykładem takiego przekazanie (Passthrough) karty graficznej do maszyny wirtualnej, żeby jej użyć np. do zdalnego grania, lub do transkodowania plików wideo.

W moim przypadku programem rozruchowym jest GRUB. Dlatego też, żeby włączyć IOMMU, musimy zmodyfikować plik gruba.

```bash
nano /etc/default/grub
```
znajdujemy linijkę 

```bash
GRUB_CMDLINE_LINUX_DEFAULT="quiet"
```

Jako, że nasz komputer jest wyposażony w procesor Intela modyfikujemy tą linijkę w sposób następujący:

```bash
GRUB_CMDLINE_LINUX_DEFAULT="quiet intel_iommu=on"
```

po zmianie - wydajemy polecenie:

```bash
update-grub
```

Po restarcie (komenda `reboot` albo wyklikane w GUI) powinniśmy mieć większy dostęp do sprzętu i mieć możliwość łatwiejszego przekazywania go do maszyn wirtualnych.

### Pierwsza Maszyna Wirtualna
OK, czas na pierwszą maszynę wirtualną (w końcu!). Na pierwszy ogień pójdzie Ubuntu 22.04.4 LTS w wersji Server. 
Przekopujemy się przez menu jak na obrazie poniżej:
![ISO upload](/assets/img/2024-04-20/iso-image.jpg)

Tutaj mamy do wyboru dwie możliwości:
* wrzucić plik ISO, który mamy juz ściągnięty na nasz komputer,
* rozkazać serwerowi na ściągnięcie pliku ISO, który znajduje się w Internecie. 
Jak plik ISO już będzie na serwerze - klikamy `Create VM` w prawym górnym rogu ekranu.

#### General
* Node - wybiera na której fizycznej maszynie ma być uruchomiona maszyna wirtualna
* VM ID - Indywidualny numer identyfikacyjny maszyny wirtualnej. Minimalna wartość to 100.
* Description - Nasz opis maszyny. Może to być cokolwiek, np. nazwa systemu operacyjnego, nazwa usług, które będzie obsługiwała, itp.

#### OS
* Storage - na razie zostaje local
* ISO image - wybieramy nasze, przed chwilą ściągnięte, ubuntu
* Type - zostaiwamy Linux
* Version - zostawiamy jak jest, 6.x - 2.6 Kernel

#### System
* Graphic Card - zostawiamy na Default
* Machine - zmieniamy na Q35
* Bios - zostaje Seafile

#### Discs
Ustawimy jeden Dysk, z interfacem SATA, o pojemności 64GiB.

#### CPU
* Sockets - zostawiamy 1
* Cores - ustawimy 2 (żeby używać 2 wątków z naszego procesora)
* Typ - zostawiamy na `x86-64-v2-AES`

#### Memory
W tej zakładce podajemy ilość pamięci, którą maksymamalnie może użyć nasza maszyna wirtualna. Ustawiamy 4096.

#### Network
W tym momencie ustawiamy wszystko na ustawieniach domyslnych.

#### Confirm
Mamy podsumowanie wszystkich danych naszej maszyny, sprawdzamy czy nie ma gdzieś błędu i klikamy `Finish`.

Kolejny sukces.
Teraz nie zostaje nam nic innego jak odpalić naszą maszynę wirtualną wybierając `Start` i rozpocząć proces instalacji systemu operacyjnego, np. wybierając połączenie VNC klikając `Console`. 
![pierwsze VM](/assets/img/2024-04-20/pierwsze-vm.jpg)

## Co dalej?
Wiele jeszcze przed nami. Pierwsze kontenery LXC, konfiguracja dodatkowych dysków i zasobów dyskowych, kolejne maszyny wirtualne.

Zapraszam niedługo!
