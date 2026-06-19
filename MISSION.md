---
# r-server — Resource Server Mission

## Mission
Homelab server for Aaryan Tahir's photo/video backup (Immich), password management (Vaultwarden), and infrastructure monitoring.

---

## Server Info
- **Hostname:** resource-server
- **Tailscale IP:** 100.84.224.18
- **OS:** Ubuntu 24.04.1 LTS (Server)
- **SSH:** `sshpass -p 'aarz1947' ssh r-server@100.84.224.18`
- **Hardware:** DDR3 32GB RAM, i5 (4 cores), 5TB HDD + 500GB SSD, **RTX 2070 8GB**
- **Storage mount:** `/mnt/storage` (2.8TB)
- **Docker compose:** `/home/r-server/docker-compose.yml` (primary, all services)
- **Immich compose:** `/home/r-server/docker/immich/docker-compose.yml` (runs separately)
- **Docker socket:** `/var/run/docker.sock`

---

## Service Status (2026-06-18)

| | Service | Container | Port | Status | Notes |
|-|---------|-----------|------|--------|-------|
| ✅ | Homepage | homepage | 8383 | UP | r-server dashboard |
| ✅ | Vaultwarden | vaultwarden | 8080 | UP | Password manager |
| ✅ | Uptime Kuma | uptime-kuma | 3001 | UP | Monitors all services |
| ✅ | Glances | glances | 61208 | UP | System stats (host pid/net) |
| ✅ | Immich | immich | 2283 | UP | Photo/video backup |
| ✅ | Immich Postgres | immich-postgres | — | UP | DB on SSD |
| ✅ | Immich Redis | immich-redis | — | UP | Valkey 9 |
| ✅ | Immich ML | immich-machine-learning | — | UP | GPU-accelerated |

**8 containers total. All UP.**

---

## Hardware Utilization (2026-06-18)
- **CPU:** i5-2500K, load 0.28, ~27°C
- **RAM:** 2.8GB used / 31GB available
- **GPU:** RTX 2070 8GB — Immich ML uses it
- **Storage:**
  - **SSD (/)** — 48GB used / 387GB free (11%)
  - **HDD (/mnt/storage)** — 7.7GB used / 2.8TB free (1%)

---

## Kuma Monitors (2026-06-18)
- **2 monitors active:** Immich · Vaultwarden
- **Fixed 2026-06-18:** DB vacuum, orphaned heartbeat cleanup, FlareSolverr ghost monitor deleted

---

## Port Access Map
```
8080   → Vaultwarden (HTTP internal, localhost only)
2283   → Immich (photo/video backup)
3001   → Uptime Kuma (monitoring dashboard)
61208  → Glances (system stats agent)
8383   → Homepage (r-server dashboard)
```

---

## Storage Layout
```
/mnt/storage/          ← 2.8TB
├── immich/
│   ├── library/        ← Original uploads (source of truth)
│   ├── encoded-video/  ← Transcoded videos
│   ├── thumbs/         ← Thumbnails
│   ├── backups/        ← Immich DB backups
│   ├── profile/        ← User profiles
│   └── model-cache/    ← ML model cache

/home/r-server/docker/       ← Compose + container configs (SSD)
/home/r-server/docker/homepage/
/home/r-server/docker/vaultwarden/
/home/r-server/docker/uptime-kuma/
/home/r-server/docker/immich-db/    ← PostgreSQL data (SSD)
/home/r-server/docker/immich-redis/ ← Valkey data (SSD)
/home/r-server/docker/immich/.env   ← Immich env vars
/home/r-server/docker/immich/docker-compose.yml  ← Immich stack (runs separately)
```

---

## Key URLs & Credentials
| Service | URL | Username | Password |
|---------|-----|----------|----------|
| Immich | http://100.84.224.18:2283 | aaryantahir8918@gmail.com | aarz1947 |
| Uptime Kuma | http://100.84.224.18:3001 | — | — |
| Homepage | http://100.84.224.18:8383 | — | — |
| Vaultwarden | http://100.84.224.18:8080 | aaryantahir8918@gmail.com | aarz1947 |

---

## What Was Removed (2026-06-17 to 2026-06-18)
- **Dashy** — redundant dashboard (2026-06-18)
- **slskd** — Soulseek P2P client (2026-06-18)
- **Jellyfin** — movies/TV server (port 8096)
- **Reclaimerr** — Jellyfin cleanup bot (port 8977)
- **Soularr** — music request manager (port 8265)
- **Prowlarr** — indexer management (port 9696)
- **Jackett** — torrent/USNet tracker proxy (port 9117)
- **qBittorrent** — torrent client (ports 6811/6881)
- **FlareSolverr** — anti-captcha (port 8191)
- **~15.5GB music files** in `/mnt/storage/downloads/`
- **Stale compose files:** `media-compose.yml`, `download-compose.yml`

---

## Known Issues / Debugging Notes
- **SSH user is `r-server@`**, not `aarz@` — `aarz@` has password auth disabled.
- **Kuma DB repair 2026-06-18:** Orphaned heartbeat records for deleted monitors purged, VACUUMed, integrity `ok`. Backup at `kuma.db.bak-*`.
- **Vaultwarden HTTP only:** Runs on internal port 8080. External HTTPS handled by nginx.
- **Immich runs from separate compose:** `/home/r-server/docker/immich/docker-compose.yml`

---

## GPU-Accel Opportunities (RTX 2070 8GB — currently used by Immich ML)
- Ollama — local LLM inference
- Stable Diffusion — AI image generation
- Whisper — speech-to-text
- Frigate NVR — camera object detection

---

## Git History
| Commit | Description |
|--------|-------------|
| `xxxxxxx` | chore: remove Dashy + slskd, update compose + MISSION to 8-container state |
| `fc18bfc` | New /mini endpoint for Dashy System Status tile |
| `d44b647` | fix: fully purge music data from r-server |
| `6bd5e1f` | refactor: split docker-compose into 4 functional groups |

---

## Mission Repo
https://github.com/Aarz-aaryan/r-server
