# Docker

Maszyna wirtualna z Debianem 13 przeznaczona na aplikacje w kontenerach. Docker Engine i Compose są zainstalowane z oficjalnego repozytorium Dockera. Każda aplikacja to osobny stos Compose w katalogu `/opt/docker/<aplikacja>` i ma własny opis w tym folderze.

## Decyzje

| Decyzja | Uzasadnienie |
|---|---|
| Docker w osobnej VM, nie w LXC | zalecenie Proxmoxa: własne jądro, bez problemów z zagnieżdżaniem kontenerów przy aktualizacjach |
| Oficjalne repozytorium Dockera | aktualny Docker Engine z wtyczką Compose; pakiety z Debiana są starsze |
| Jeden katalog na aplikację w `/opt/docker` | plik Compose, `.env` i dane w jednym miejscu – łatwy backup i przeniesienie |
| Hasła w `.env` | plik nie trafia do repozytorium, w repo jest tylko `.env.example` |
| Konkretne wersje obrazów zamiast `latest` | aktualizacja następuje wtedy, gdy ją zaplanuję, a nie przy przypadkowym `pull` |
| Rotacja logów kontenerów | logi nie zapełnią dysku maszyny |

## 1. Maszyna wirtualna

| Parametr | Wartość |
|---|---|
| System | Debian 13 (netinst), bez środowiska graficznego, z serwerem SSH |
| CPU | 2 rdzenie, typ `host` |
| RAM | 2 GB, minimum 1 GB (ballooning) |
| Dysk | 32 GB na `local-lvm`, VirtIO SCSI single, włączone Discard i SSD emulation |
| Sieć | VirtIO na `vmbr0`, statycznie 192.168.1.52/24, brama 192.168.1.1 |
| Agent QEMU | włączony |
| Autostart | wyłączony – maszyna uruchamiana w razie potrzeby |



Agent QEMU pozwala Proxmoxowi poprawnie wyłączać maszynę i robić spójne kopie zapasowe. W Proxmoxie jest włączony przy tworzeniu maszyny, a w systemie trzeba go zainstalować:

```bash
apt install -y qemu-guest-agent
systemctl enable --now qemu-guest-agent
```

## 2. Instalacja Dockera

Repozytorium Dockera:

```bash
apt update
apt install -y ca-certificates curl
install -m 0755 -d /etc/apt/keyrings
curl -fsSL https://download.docker.com/linux/debian/gpg -o /etc/apt/keyrings/docker.asc
chmod a+r /etc/apt/keyrings/docker.asc

tee /etc/apt/sources.list.d/docker.sources <<EOF
Types: deb
URIs: https://download.docker.com/linux/debian
Suites: $(. /etc/os-release && echo "$VERSION_CODENAME")
Components: stable
Signed-By: /etc/apt/keyrings/docker.asc
EOF
```

Pakiety:

```bash
apt update
apt install -y docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin
docker run --rm hello-world
```

## 3. Konfiguracja

Rotacja logów – każdy kontener trzyma najwyżej 3 pliki po 10 MB. Ustawienie dotyczy kontenerów tworzonych od tej chwili:

```bash
cat > /etc/docker/daemon.json <<'EOF'
{
  "log-driver": "json-file",
  "log-opts": { "max-size": "10m", "max-file": "3" }
}
EOF
systemctl restart docker
```

Praca bez `sudo` dla użytkownika administracyjnego (po ponownym zalogowaniu):

```bash
usermod -aG docker <użytkownik>
```

Członkostwo w grupie `docker` daje w praktyce uprawnienia roota, więc trafia do niej tylko konto administratora.

## 4. Układ aplikacji

```
/opt/docker/
└── <aplikacja>/
    ├── docker-compose.yml   # definicja stosu
    ├── .env                 # hasła i wersje obrazów, chmod 600, poza repozytorium
    └── <dane>/              # katalogi montowane do kontenerów
```

Zasady dla każdego stosu:
- wersje obrazów zapisane w `.env` albo wprost w pliku Compose, bez tagu `latest`
- `restart: unless-stopped`, żeby kontenery wstawały po restarcie maszyny
- na zewnątrz publikowany jest tylko port aplikacji; bazy danych i cache działają wyłącznie w wewnętrznej sieci stosu
- dane w podkatalogach obok pliku Compose, dzięki czemu cały stos to jeden katalog

Podstawowe polecenia, wykonywane w katalogu stosu:

```bash
docker compose up -d          # uruchomienie lub zastosowanie zmian
docker compose ps             # stan kontenerów
docker compose logs -f        # logi na żywo
docker compose down           # zatrzymanie (dane zostają)
```

## 5. Utrzymanie

**Aktualizacja Dockera** odbywa się razem z systemem (`apt update && apt upgrade`), bo pakiety pochodzą z repozytorium.

**Aktualizacja obrazów** aplikacji, po zmianie wersji w `.env` lub pliku Compose:

```bash
cd /opt/docker/<aplikacja>
docker compose pull
docker compose up -d
docker image prune -f
```








