# RustDesk Server mit API (s6-Image)

Selbst gehosteter [RustDesk](https://rustdesk.com)-Server (ID-Server **hbbs** und Relay **hbbr**) zusammen mit der [RustDesk-API](https://github.com/MrMarkus42/rustdesk-api) und deren Admin-Oberfläche in **einem** Docker-Container.

Dieser Fork baut auf [lejianwen/rustdesk-server](https://github.com/lejianwen/rustdesk-server) auf (Branch `forapi`), der nicht mehr gepflegt wird, und führt ihn mit dem aktuellen offiziellen [rustdesk/rustdesk-server](https://github.com/rustdesk/rustdesk-server) zusammen.

## Funktionen

- **Login-Pflicht** für Verbindungen (`MUST_LOGIN=Y`), Prüfung des Logins per JWT
- Anmeldung der Geräte auch über **TCP und WebSocket** (Clients ≥ 1.4.1), verschlüsselte TCP-Verbindung zum Client
- **API mit Admin-Oberfläche** (Deutsch/Englisch): Benutzer, Adressbücher, Gruppen, Geräte, OIDC/LDAP, Verbindungs- und Dateiprotokolle
- Stand des offiziellen Servers **1.1.17** inkl. aller Sicherheitskorrekturen (u. a. Schutz gegen UDP-Amplification)

### Änderungen gegenüber lejianwen

- Aktueller offizieller Server eingearbeitet
- Server stürzt bei fehlerhaftem Schlüsselaustausch nicht mehr ab
- `-k <öffentlicher Schlüssel>` lädt den passenden privaten Schlüssel, Warnung bei einfachen Keys (mit Clients ≥ 1.4.1 nicht nutzbar)
- Tägliche Update-Abfrage an `api.rustdesk.com` abschaltbar (`UPDATE_CHECK=N`)
- Image baut auf der gepflegten API [MrMarkus42/rustdesk-api](https://github.com/MrMarkus42/rustdesk-api) auf (Sicherheitskorrekturen, deutsche Oberfläche)

## Docker-Image

```
ghcr.io/mrmarkus42/rustdesk-server-s6:<version>
```

Architekturen: `amd64`, `arm64`, `armv7`. Für den Produktivbetrieb eine feste Version statt `latest` verwenden.

### Beispiel (docker compose)

```yaml
services:
  rustdesk:
    image: ghcr.io/mrmarkus42/rustdesk-server-s6:1.1.17-2
    network_mode: host            # oder Ports 21114–21119 (21116 auch UDP) freigeben
    environment:
      - TZ=Europe/Berlin
      - RELAY=rustdesk.example.org
      - ENCRYPTED_ONLY=1
      - MUST_LOGIN=N
      - RUSTDESK_API_RUSTDESK_ID_SERVER=rustdesk.example.org
      - RUSTDESK_API_RUSTDESK_RELAY_SERVER=rustdesk.example.org
      - RUSTDESK_API_RUSTDESK_API_SERVER=https://rustdesk.example.org
      - RUSTDESK_API_JWT_KEY=<langer zufälliger Wert>
      - RUSTDESK_API_APP_WEB_CLIENT=0
    volumes:
      - ./data/server:/data       # Server-Schlüssel und Geräte-DB, unbedingt persistent!
      - ./data/api:/app/data      # API-Datenbank
    restart: unless-stopped
```

Das initiale Admin-Passwort steht beim ersten Start im Log. Die Admin-Oberfläche liegt unter `/_admin/`.

## Wichtige Einstellungen

| Variable | Bedeutung |
|---|---|
| `RELAY` | Adresse des Relay-Servers für die Clients |
| `ENCRYPTED_ONLY=1` | Nur verschlüsselte Verbindungen, nutzt das Schlüsselpaar aus `/data` |
| `MUST_LOGIN=Y` | Verbindungen nur mit angemeldetem Client |
| `RUSTDESK_API_JWT_KEY` | Signatur der Login-Tokens. **Nach der Einrichtung nicht mehr ändern**, sonst müssen sich alle Clients neu anmelden |
| `RUSTDESK_API_APP_WEB_CLIENT=0` | Web-Client abschalten |
| `RUSTDESK_API_GIN_TRUST_PROXY` | Adresse/Bereich des Reverse Proxys, damit die API die echten Client-IPs sieht (z. B. `172.16.0.0/12` für einen Proxy in Docker) |
| `RUSTDESK_API_APP_BAN_THRESHOLD=10` | IP-Sperre nach Fehlversuchen. **Nur zusammen mit korrektem `TRUST_PROXY`** |
| `UPDATE_CHECK=N` | Keine tägliche Update-Abfrage an api.rustdesk.com |

Alle weiteren Server-Parameter: [docs/environment-variables.md](docs/environment-variables.md). Alle API-Parameter: [RustDesk-API](https://github.com/MrMarkus42/rustdesk-api).

## Sicherheitshinweise

- Die API (Port 21114) nur über einen HTTPS-Reverse-Proxy erreichbar machen, nicht direkt ins Internet freigeben.
- Den Admin-Bereich (`/_admin`, `/api/admin`) möglichst auf das eigene Netz oder VPN beschränken. Eine Zwei-Faktor-Anmeldung gibt es nicht.
- `/data` (enthält `id_ed25519`) und `/app/data` regelmäßig sichern. Ein neuer Schlüssel trennt alle Clients.

## Herkunft und Lizenz

- Offizieller Server: [rustdesk/rustdesk-server](https://github.com/rustdesk/rustdesk-server)
- API-Erweiterungen: [lejianwen/rustdesk-server](https://github.com/lejianwen/rustdesk-server) ([ursprüngliche Beschreibung](README-lejianwen.md))
- Lizenz: [AGPL-3.0](LICENSE)

Weitere Sprachen der ursprünglichen Server-Dokumentation: [English](README-EN.md) · [Nederlands](README-NL.md) · [繁體中文](README-TW.md) · [Deutsch (alt)](README-DE.md)
