[Deutsch](README.md) · **English**

# RustDesk Server with API (s6 image)

Self-hosted [RustDesk](https://rustdesk.com) server (ID server **hbbs** and relay **hbbr**) together with the [RustDesk API](https://github.com/MrMarkus42/rustdesk-api) and its admin interface in **one** Docker container.

This fork is based on [lejianwen/rustdesk-server](https://github.com/lejianwen/rustdesk-server) (branch `forapi`), which is no longer maintained, merged with the current official [rustdesk/rustdesk-server](https://github.com/rustdesk/rustdesk-server).

## Features

- **Mandatory login** for connections (`MUST_LOGIN=Y`), login verified via JWT
- Device registration over **TCP and WebSocket** as well (clients ≥ 1.4.1), encrypted TCP connection to the client
- **API with admin interface** (German/English): users, address books, groups, devices, OIDC/LDAP, connection and file logs
- Official server **1.1.17** including all security fixes (e.g. protection against UDP amplification)

### Changes compared to lejianwen

- Current official server merged in
- Server no longer crashes on a malformed key exchange
- `-k <public key>` loads the matching private key; warning for simple keys (not usable with clients ≥ 1.4.1)
- Daily update check against `api.rustdesk.com` can be disabled (`UPDATE_CHECK=N`)
- Image is built on top of the maintained API [MrMarkus42/rustdesk-api](https://github.com/MrMarkus42/rustdesk-api) (security fixes, German UI)

## Docker image

```
ghcr.io/mrmarkus42/rustdesk-server-s6:<version>
```

Architectures: `amd64`, `arm64`, `armv7`. Use a fixed version instead of `latest` in production.

### Example (docker compose)

```yaml
services:
  rustdesk:
    image: ghcr.io/mrmarkus42/rustdesk-server-s6:<version>
    network_mode: host            # or publish ports 21114–21119 (21116 also UDP)
    environment:
      - TZ=Europe/Berlin
      - RELAY=rustdesk.example.org
      - ENCRYPTED_ONLY=1
      - MUST_LOGIN=N
      - RUSTDESK_API_RUSTDESK_ID_SERVER=rustdesk.example.org
      - RUSTDESK_API_RUSTDESK_RELAY_SERVER=rustdesk.example.org
      - RUSTDESK_API_RUSTDESK_API_SERVER=https://rustdesk.example.org
      - RUSTDESK_API_JWT_KEY=<long random value>
    volumes:
      - ./data/server:/data       # server keys and device DB, must be persistent!
      - ./data/api:/app/data      # API database
    restart: unless-stopped
```

The initial admin password is printed to the log on first start. The admin interface is at `/_admin/`.

## Important settings

| Variable | Meaning |
|---|---|
| `RELAY` | Relay server address handed to clients |
| `ENCRYPTED_ONLY=1` | Encrypted connections only, uses the key pair in `/data` |
| `MUST_LOGIN=Y` | Connections only from logged-in clients |
| `RUSTDESK_API_JWT_KEY` | Signs the login tokens. **Do not change it after setup**, otherwise all clients have to log in again |
| `RUSTDESK_API_GIN_TRUST_PROXY` | Address/range of the reverse proxy so the API sees the real client IPs (e.g. `172.16.0.0/12` for a proxy running in Docker) |
| `RUSTDESK_API_APP_BAN_THRESHOLD=10` | IP ban after failed attempts. **Only together with a correct `TRUST_PROXY`** |
| `UPDATE_CHECK=N` | No daily update check against api.rustdesk.com |

All other server options: [docs/environment-variables.md](docs/environment-variables.md). All API options: [RustDesk API](https://github.com/MrMarkus42/rustdesk-api).

## Security notes

- Expose the API (port 21114) only through an HTTPS reverse proxy, never directly to the internet.
- Restrict the admin area (`/_admin`, `/api/admin`) to your own network or VPN if possible. There is no two-factor login.
- Back up `/data` (contains `id_ed25519`) and `/app/data` regularly. A new key disconnects all clients.

## Origin and license

- Official server: [rustdesk/rustdesk-server](https://github.com/rustdesk/rustdesk-server)
- API extensions: [lejianwen/rustdesk-server](https://github.com/lejianwen/rustdesk-server) ([original description, Chinese](README-lejianwen.md))
- License: [AGPL-3.0](LICENSE)

Original server documentation: [English](README-EN.md) · [Deutsch](README-DE.md) · [Nederlands](README-NL.md) · [繁體中文](README-TW.md)
