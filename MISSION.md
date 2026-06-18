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
- **Docker:** Single `docker-compose.yml` at `/home/r-server/docker/` — all services defined here
- **Docker socket:** `/var/run/docker.sock`

---

## Service Status (2026-06-17)

| | Service | Container | Port | Status | Notes |
|-|---------|-----------|------|--------|-------|
| ✅ | Immich | immich-server | 2283 | UP | Photo/video backup — **RTX 2070 for ML** |
| ✅ | Immich Postgres | immich-postgres | — | UP | DB on SSD |
| ✅ | Immich Redis | immich-redis | — | UP | Healthy |
| ✅ | Immich ML | immich-machine-learning | — | UP | GPU-accelerated |
| ✅ | Uptime Kuma | uptime-kuma | 3001 | UP | Monitors all services |
| ✅ | Homepage | homepage | 8383 | UP | r-server dashboard |
| ✅ | Dashy | dashy | 4380 | UP | Backup dashboard |
| ✅ | Glances | glances | 61208 | UP | System monitoring agent |
| ✅ | Vaultwarden | vaultwarden | 8080 | UP | Password manager |
| ✅ | slskd | slskd | 5030-5031 | UP | Soulseek client |

**10 containers total. All UP.**

**Removed 2026-06-17:** Jellyfin, Reclaimerr, Soularr, Prowlarr, Jackett, qBittorrent, FlareSolverr — full media stack nuked.

---

## Hardware Utilization (2026-06-17)
- **CPU:** Load ~0.12, 4 cores
- **RAM:** 5GB used / 31GB available
- **GPU:** RTX 2070 8GB — Immich ML uses it
- **Storage:**
  - **SSD (/)** — 31GB used / 404GB free (7%)
  - **HDD (/mnt/storage)** — 7.7GB used / 2.8TB free (1%)

---

## Kuma Monitors (2026-06-17)
- **2 monitors active:** Immich · Vaultwarden
- **Removed 2026-06-17:** Jellyfin, qBittorrent, Jackett, Prowlarr, FlareSolverr

---

## Port Access Map
```
8080   → Vaultwarden (HTTP internal)
2283   → Immich (photo/video backup)
3001   → Uptime Kuma (monitoring dashboard)
4380   → Dashy (backup dashboard)
5030-5031 → slskd (Soulseek client)
61208  → Glances (system stats agent)
8383   → Homepage (r-server dashboard)
```

---

## Storage Layout
```
/mnt/storage/          ← 2.8TB
├── immich/           ← Photos/videos (7.1GB)
│   ├── library/      ← Original uploads (source of truth)
│   ├── encoded-video/← Transcoded videos
│   ├── thumbs/       ← Thumbnails
│   ├── backups/      ← Immich DB backups
│   ├── profile/      ← User profiles
│   └── model-cache/  ← ML model cache
├── immich-db/        ← PostgreSQL data
└── slskd/            ← Soulseek data

/home/r-server/docker/  ← Compose + container configs (SSD)
/home/r-server/docker/immich-db/  ← DB on SSD
```

---

## Key URLs & Credentials
| Service | URL | Username | Password |
|---------|-----|----------|----------|
| Immich | http://100.84.224.18:2283 | aaryantahir8918@gmail.com | aarz1947 |
| Uptime Kuma | http://100.84.224.18:3001 | — | — |
| Homepage | http://100.84.224.18:8383 | — | — |
| Vaultwarden | http://100.84.224.18:8080 | aaryantahir8918@gmail.com | aarz1947 |
| Dashy | http://100.84.224.18:4380 | — | — |

---

## What Was Removed (2026-06-17)
- **Jellyfin** — movies/TV server (port 8096)
- **Reclaimerr** — Jellyfin cleanup bot (port 8977)
- **Soularr** — music request manager (port 8265)
- **Prowlarr** — indexer management (port 9696)
- **Jackett** — torrent/USNet tracker proxy (port 9117)
- **qBittorrent** — torrent client (ports 6811/6881)
- **FlareSolverr** — anti-captcha (port 8191)
- **~15.5GB music files** in `/mnt/storage/downloads/`
- **Stale compose files:** `media-compose.yml`, `download-compose.yml`
- **Kuma monitors** for all removed services

---

## Known Issues / Debugging Notes
- **SSH user is `r-server@`**, not `aarz@` — `aarz@` has password auth disabled.
- **Kuma API auth:** Direct SQLite query via SSH is the reliable method.
- **Vaultwarden HTTP only:** Runs on internal port 8080. External HTTPS handled by nginx.

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
| `fc18bfc` | New /mini endpoint for Dashy System Status tile |
| `d44b647` | fix: fully purge music data from r-server |
| `6bd5e1f` | refactor: split docker-compose into 4 functional groups |

---

## Mission Repo
https://github.com/Aarz-aaryan/r-server
---