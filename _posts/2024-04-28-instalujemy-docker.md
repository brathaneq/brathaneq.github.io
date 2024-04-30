---
title: Instalujemy docker na VM
author: brat
date: 2024-04-28 00:00:00 +0000
categories: [Homelab]
tags: [software, proxmox, ubuntu, docker]
render_with_liquid: false
---
## Czas na kontenery!

Cześć!
Niedawno pojawiłą się informacja o wypuszczeniu nowej wersji Ubuntu - 24.04 Noble Numbat. Po aktualizacji pomyślałem, że to dobry moment, żeby zacząć w końcu z niej korzystać. Dlatego też w dzisiejszym wpisie opiszemy instalację dockera na systemie Ubuntu.


### Instalacja

Oczywiście nie wymyślimy tutaj koła od nowa, będziemy się opierać na [oficjalnej](https://docs.docker.com/engine/install/ubuntu/) dokumentacji. 

#### Przygotowanie repozytorium  `apt` dockera.
Logujemy się na nasz serwer, czy to przy pomocy ssh, czy to z użyciem konsoli w Proxmoxie i po kolei wpisujemy poniższe komendy:

```bash
# Dodanie oficjalnego klucza GPG :
sudo apt-get update
sudo apt-get install ca-certificates curl gnupg
sudo install -m 0755 -d /etc/apt/keyrings
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /etc/apt/keyrings/docker.gpg
sudo chmod a+r /etc/apt/keyrings/docker.gpg

# Dodanie repozytorium do źródeł apt:
echo \
  "deb [arch="$(dpkg --print-architecture)" signed-by=/etc/apt/keyrings/docker.gpg] https://download.docker.com/linux/ubuntu \
  "$(. /etc/os-release && echo "$VERSION_CODENAME")" stable" | \
  sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
sudo apt-get update
```

#### Instalujemy najnowszą wersję dockera.
System jest już przygotowowany do instalacji, więc przed nami kolejne kroki:

```bash
sudo apt-get install docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin
```
{: .nolineno }

#### Sprawdzmy która wersja jest zainstalowana.

```bash
docker -v
```
{: .nolineno }

#### Sprawdzmy czy compose jest zainstalowane poprawnie.

```bash
docker compose
```
{: .nolineno }

#### Pierwsze uruchomienie

```bash
 sudo docker run hello-world
```
{: .nolineno }

Jeżeli powyższe uruchomi się bez błędów - gratulacje, masz w pełni działajacego dockera!

### Używanie dockera bez komendy sudo
Niedługo będziemy sporo używali komendy docker, która wymaga uprawnień root. Żeby ułatwić sobie życie - skonfigurujmy system tak, żeby nie trzeba było za każdym razem wpisywać "sudo" przed komendami. 

```bash
sudo usermod -aG docker $USER
```
{: .nolineno }

Po powyższej komendzie należy się przelogować.

## Co dalej?
Przed nami migracja usług z aktualnego serwera na nowy. Raczej będziemy wszystko konfigurować od nowa, choć szybciej pewnie by było "odzyskiwać" ustawienia z backupów. 
Zapraszam niedługo!
