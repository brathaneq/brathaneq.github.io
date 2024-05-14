---
title: Instalujemy VS Code
author: brat
date: 2024-05-15 00:00:00 +0000
categories: [Homelab]
tags: [software, proxmox, ubuntu, docker, vscode, traefik, authentik, crowdsec, vaultwarden, cloudflare]
render_with_liquid: false
---
## Czas na nasz pierwszy kontener.

Cześć!
Mamy na naszym serwerze zainstalowany już docker, czas rozpocząć wrzucać pierwsze kontenery z usługami. Będziemy je "migrować" z mojego aktualnego serwera, ale na potrzeby tego bloga, zamiast odzyskiwać konfiguracje, będziemy wszystko konfigurować od zera, i jeżeli tylko będzie taka możliwość, wrzucimy konfigurację na [githuba](https://github.com/brathaneq/docker).

### Code Server
Zaczniemy nietypowo, bo od Visual Studio Code. Jest to edytor kodu, który obsługujemy przez przeglądarkę internetową. Edytor obsługuje wiele dodatków, np. sprawdzających składnię m.in.  w plikach yaml, markdown. Ma wewnętrzną obsługę Git, połączeń SSH. Moglibyśmy oczywiście pominąć ten krok i korzystać z wbudowanego edytora tekstu, np nano. Sam lubię go używać do prostych i szybkich zmian, jednak przy pracy która nas czeka - warto zainwestować dodatkową chwilę i zainstalować VS Code.

#### Różne możliwości instalacji, w naszym przypadku docker
Na oficjalnej [stronie projektu](https://coder.com/docs/code-server/latest/) znaleźć można wiele sposobów [instalacji](https://coder.com/docs/code-server/latest/install), od różnych dystrybucji linuxa, przez Windowsa, kończąc na dostawcach usług w chmurze. W naszym przypadku, na maszynie wirtualnej, będziemy wrzucać kontener, którym będziemy mogli przygotować i modyfikować pliki konfiguracyjne następnych kontenerów. 

#### Instalacja krok po kroku
Na oficjalnej stronie, wspomnianej wyżej, mamy gotowy do wklejenia kawałek kodu, który uruchomi nam kontener, jednak ze względu na czytelność i wygodę użyjemy docker compose. Na stronie [LinuxServer.io](https://docs.linuxserver.io/images/docker-code-server/) możemy znaleźć dodatkową dokumentację, wraz z przykładowym pliczkiem compose. Jest on też skopiowany na moim [githubie](https://github.com/brathaneq/docker), ale pewnie najnowszą wersję znajdziecie jednak na LinuxServer.io. 

##### Przygotowanie miejsca pod kontenery
Logujemy się na serwer (czy to przy pomocy ssh, czy też odpalając konsolę w PVE) i zakładamy katalog w którym nasze kontenery będą trzymać swoje dane. W moim przypadku będzie to `/opt/appdata`{: .filepath}, ale można to też zrobić np. w katalogu `/home/$USER`{: .filepath}.

```bash
cd /opt
sudo mkdir appdata
cd appdata
sudo mkdir code-server
cd code-server
```
{: .nolineno }

Teraz trzeba wkleić konfigurację do pliku `docker-compose.yaml`

```yaml
---
services:
  code-server:
    image: lscr.io/linuxserver/code-server:latest
    container_name: code-server
    environment:
      - PUID=1000
      - PGID=1000
      - TZ=Etc/UTC
      - PASSWORD=password #optional
      - HASHED_PASSWORD= #optional
      - SUDO_PASSWORD=password #optional
      - SUDO_PASSWORD_HASH= #optional
      - PROXY_DOMAIN=code-server.my.domain #optional
      - DEFAULT_WORKSPACE=/config/workspace #optional
    volumes:
      - /path/to/appdata/config:/config
    ports:
      - 8443:8443
    restart: unless-stopped
```
{: file="/opt/appdata/code-server/docker-compose.yaml" }

Robimy kilka modyfikacji:
* w linijce TZ - wpisujemy Europe/Warsaw
* docelowo w HASHED_PASSWORD wkleimy hash naszego hasła, na razie, przy pierwszym uruchomieniu uzupełnimy linię PASSWORD=
* proxy domain na razie poprzedzamy znakiem #
* możemy dodać dodatkowy wpis w volumes, żeby udostępnić sobie inny katalog, np `/opt`, albo `/home`
* możemy skorygować ścieżkę w `DEFAULT_WORKSPACE` żeby kierowało nas od razu do odpowiedniego katalogu

po zapisaniu pliku uruchamiamy

```bash
docker compose up -d
```
{: .nolineno }

Powinniśmy zobaczyć postęp w ściąganiu kontenera, po czym naszym oczom powinien się pojawić komunikat 
```
✔ Container code-server        Started 
```
sprawdzmy, czy wszystko jest OK przy użyciu komendy `docker ps`.

##### połączenie się i konfiguracja pierwszych pluginów
Teraz możemy się zalogować na Code-Server wchodząc na adres naszego serwera, specyfikując port 8443.
Sukces!
Na ekranie powinna się pojawić prosba o wpisanie hasła:

![code-login](/assets/img/2024-05-10/code-login.jpg)

##### Instalacja rozszerzeń
Na początku pracy, dla naszej wygody, zainstalujemy kilka rozszerzeń. Żeby to zrobić - wybieramy ikonę rozszerzeń z menu znajdującego się po lewej stronie, lub używamy skrótu klawiszowego `Ctrl+Shift+X) i używamy pola wyszukiwania które się pojawi.
Interesuje nas m.in:
* [Markdown All in One] (https://open-vsx.org/extension/yzhang/markdown-all-in-one) - dodatek pomagający pisać kod w języku Markdown
* [YAML](https://open-vsx.org/extension/redhat/vscode-yaml) - jak wyżej, tylko pomaga w kodowaniu z użyciem yaml
*  [VSCode Great Icons](https://open-vsx.org/extension/emmanuelbeziat/vscode-great-icons) - dodatek dodający czytelne ikony przy plikach. 
*  [SSH FS](https://open-vsx.org/extension/Kelvin/vscode-sshfs) - dodatek pozwalający na zdalną modyfikację plików oraz uruchamianie terminalu na zdalnym serwerze.
  
##### SSH FS - obsługa połączeń + modyfikacja plików
Dodatek znajdziemy po lewej stronie. Po jego wyborze kliknięciem - mamy 2 okna do przeprowadzenia konfiguracji:
![sshfs](/assets/img/2024-05-10/sshfs.jpg)

Zaczynamy od górnego okna - po najechaniu na `CONFIGURATIONS` pojawią nam się dodatkowe ikony i tam wybieramy `Create a SSH FS Configuration`

![sshfs-config](/assets/img/2024-05-10/sshfs-config.jpg)

Po kolei przechodzimy przez konfiguracje (wpisujemy adresy, loginy, etc). W górnym oknie w polu `Connections` pojawi się nam nasze połaczenie.
Po najechaniu na nie - możemy wybrać `Open remote SSH terminal`.

*Sukces!*

Powinno nam się pojawić okno z terminalem do pracy na zdalnej maszynie.
Dodatkowo - po najechaniu na połączenie - mamy opcję `Add as a Workspace folder`. Po jego wyborze - w Explorerze pojawi nam się dodatkowy obszar, opisany jako `SSH FS - nazwa_naszego_połączenia`. Tam znajdziemy wszystkie pliki do których mamy dostęp na zdalnym serwerze.

#### Podsumowanie
Udało nam się zainstalować i skonfigurować podstawy w VS Code. Dzięki temu, użyciu jednego narzędzia, możemy m.in:
* modyfikować pliki konfiguracyjne na kilku serwerach jednocześnie
* zdalnie zarządzać przez ssh serwerami
* wgrywać i ściągać pliki na serwery
* synchronizować z githubem

### Co dalej ? 
Przed nami jeszcze sporo pracy, powinniśmy zwiększyc tempo. Przed nami m.in. reverse proxy (w moim przypadku będzie to [Traefik](https://doc.traefik.io/traefik/)), manager haseł [Vaultwarden](https://github.com/dani-garcia/vaultwarden/wiki]), serwer mediów (u mnie [Plex](https://www.plex.tv/)). Zapraszam do następnych wpisów.

