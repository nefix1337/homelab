# Homelab - dokumetnacja 

![Proxmox VE](https://img.shields.io/badge/Proxmox_VE-E57000?logo=proxmox&logoColor=white)
![Windows Server 2022](https://img.shields.io/badge/Windows_Server_2022-0078D4)
![PowerShell](https://img.shields.io/badge/PowerShell-5391FE)
![WireGuard](https://img.shields.io/badge/WireGuard-88171A?logo=wireguard&logoColor=white)
![Docker](https://img.shields.io/badge/Docker-2496ED?logo=docker&logoColor=white)
![Zabbix](https://img.shields.io/badge/Zabbix-D40000)
![GLPI](https://img.shields.io/badge/GLPI-002F6C)

**Status: w trakcie tworzenia**

# Założenia:
Domowe środowisko do eksperymentów, w którym rozwijam umiejętności z zakresu administracji systemami, sieci i bezpieczeństwa. Odtwarzam w nim podstawowe elementy infrastruktury IT: domenę Active Directory, zdalny dostęp przez VPN, monitoring oraz system helpdesk z ewidencją sprzętu.


## Zakres

### Wirtualizacja (Proxmox VE)

- Maszyny wirtualne i kontenery LXC, aplikacje w Dockerze na osobnej maszynie z Debianem
- Kopie zapasowe maszyn na osobny dysk

### Windows Server 2022 Core i Active Directory

- Instalacja w wersji Core, bez interfejsu graficznego
- Konfiguracja i administracja wyłącznie z konsoli i PowerShell
- Kontroler domeny z rolą DNS i DHCP
- Zarządzanie użytkownikami, grupami i jednostkami organizacyjnymi (OU), zasady grupy (GPO)
- Tworznie skryptów do automatyzaji i zarządzania

### Zdalny dostęp (WireGuard)

- Łącze domowe jest za CGNAT, więc rolę huba pełni VPS, a brama w laboratorium łączy się z nim wychodząco
- Na domowym routerze nie jest otwarty żaden port
- Panele administracyjne (Proxmox, Zabbix, GLPI) są dostępne przez VPN

### Monitoring (Zabbix)

- Agenty Zabbix na maszynach z Windows i Linuksem
- Podstawowe alerty: dostępność hostów, zajętość dysków, obciążenie

### Helpdesk i ewidencja sprzętu (GLPI)

- Obsługa zgłoszeń: kategorie, priorytety, przypisywanie do osób
- Ewidencja sprzętu i oprogramowania



## Architektura

...



### Sprzęt

| Element | Szczegóły |
|---|---|
| Procesor | Intel Core i3-8100 (4 rdzenie) |
| Pamięć | 8 GB DDR4 |
| SSD 256 GB | system, dyski maszyn wirtualnych i kontenerów |
| HDD 1 TB | kopie zapasowe, obrazy ISO |
| Łącze | światłowód Orange: ONT + router Funbox 3 |
| VPS | hub WireGuard |