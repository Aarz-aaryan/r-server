# r-server — Resource Server Mission

## Mission
Homelab server for Aaryan Tahir's media management, photo/video backup, password management, and infrastructure monitoring.

---

## Server Info
- **Hostname:** resource-server
- **Tailscale IP:** 100.84.224.18
- **OS:** Ubuntu 24.04.1 LTS (Server)
- **SSH:** `sshpass -p 'aarz1947' ssh r-server@100.84.224.18`
- **Hardware:** DDR3 32GB RAM, i5 (4 cores), 5TB HDD + 500GB SSD, **RTX 2070 8GB**
- **Storage mount:** `/mnt/storage` (2.8TB, exFAT — special char renaming blocked)
- **Docker compose structure** (split by function):
  - `media-compose.yml` — Jellyfin
  - `download-compose.yml` — qBittorrent, Jackett, Prowlarr, FlareSolverr
  - `homelab-compose.yml` — Glances, Dashy, Uptime Kuma
  - `immich/docker-compose.yml` — Immich stack (separate project)
  - `vaultwarden/` — Vaultwarden (separate directory)
- **Docker socket:** `/var/run/docker.sock`

---

## Service Status (2026-06-12)

| | Service | Container | Port | Status | Notes |
|-|---------|-----------|------|--------|-------|
| ✅ | Jellyfin | jellyfin | 8096 | UP 2 days | Movies + TV |
| ✅ | Immich | immich | **2283** | UP 44 min | Photo/video backup, v2.7.5 |
| ✅ | Immich Postgres | immich-postgres | — | UP 2 days | PostgreSQL 16 |
| ✅ | Immich Redis | immich-redis | — | UP 2 days | Valkey 9 |
| ✅ | Immich ML | immich-machine-learning | — | UP 2 days | Healthy |
| ✅ | Uptime Kuma | uptime-kuma | 3001 | UP 2 days | Monitors all services |
| ✅ | Jackett | jackett | 9117 | UP 2 days | Torznab, FlareSolverr configured |
| ✅ | Prowlarr | prowlarr | 9696 | UP 2 days | Indexer management |
| ✅ | qBittorrent | qbittorrent | 6811 | UP 2 days | Torrent downloads |
| ✅ | Dashy | dashy | 4380 | UP 2 days | Dashboard |
| ✅ | FlareSolverr | flaresolverr | 8191 | UP 2 days | Cloudflare bypass |
| ✅ | Glances | glances | 61208 | UP 2 days | System monitoring agent |
| ✅ | Vaultwarden | vaultwarden | 80 | UP 2 days | Password manager |

| ✅ | Homepage | homepage | 8383 | UP | Dashboard — modern alternative to Dashy |
| ✅ | Reclaimerr | reclaimerr | 8977 | UP | Media library cleanup — Jellyfin/Plex/Emby |

**15 containers total. All UP.**

---

## Hardware Utilization
- **CPU:** 4 cores, ~4.4GB RAM used, ~26GB available
- **GPU:** RTX 2070 8GB — 7.9GB free (not currently in use by any container)
- **Storage:** 2.8TB mount — 1% used (3.7GB used, 2.8TB free)

---

## Dashy Dashboard
- **URL:** http://100.84.224.18:4380
- **Config:** `/home/r-server/docker/dashy/conf.yml`
- **3 sections:** Media (Jellyfin, Immich), Downloads & Indexers, System & Monitoring

### System Status Widget
Live stats via `http://100.84.224.18:9400/status` (stats-server.py + Glances API + Kuma DB).

---

## Port Access Map
```
80    → Vaultwarden (password manager)
8096  → Jellyfin (movies + TV)
2283  → Immich (photo/video backup)
8383  → Homepage (dashboard)
8977  → Reclaimerr (media cleanup)
9117  → Jackett (Torznab API)
9696  → Prowlarr (indexer management)
6811  → qBittorrent (torrent downloads)
3001  → Uptime Kuma (monitoring dashboard)
4380  → Dashy (self-hosted dashboard)
8191  → FlareSolverr (Cloudflare bypass)
61208 → Glances (system stats agent)
9400  → Stats server (Glances + Kuma JSON, bearer auth)
```

---

## Storage Layout
```
/mnt/storage/
├── media/          ← Movies + TV (Jellyfin library)
├── photos/         ← Immich photo uploads
├── downloads/
│   ├── incomplete/
│   └── completed/  ← qBittorrent torrent imports
├── nas/            ← General network storage / Samba share
└── samba/
```

---

## Key URLs & Credentials
| Service | URL | Username | Password |
|---------|-----|----------|----------|
| Jellyfin | http://100.84.224.18:8096 | aarz | aarz1947 |
| Immich | http://100.84.224.18:2283 | aarz@aarz.com | aarz1947 |
| Jackett | http://100.84.224.18:9117 | — | API: `gc4a3zfeircm6chs6tsvlen48zsw14lj` |
| Prowlarr | http://100.84.224.18:9696 | aarz | aarz1947 |
| qBittorrent | http://100.84.224.18:6811 | admin | aarz1947 |
| Uptime Kuma | http://100.84.224.18:3001 | — | — |
| Dashy | http://100.84.224.18:4380 | — | — |
| Vaultwarden | http://100.84.224.18:80 | aarz@aarz.com | aarz1947 |

---

## What Else Can We Host — Research (2026-06-12)

### GPU-Accelerated (RTX 2070 8GB — currently idle)
| # | Service | What it does | Footprint | Fit |
|---|---------|-------------|-----------|-----|
| 1 | **Ollama** | Local LLM inference (Llama, Mistral, DeepSeek) w/ GPU acceleration | Medium-Heavy | Perfect — privacy-first AI on idle GPU |
| 2 | **Open WebUI** | Web UI for Ollama — chat, model management, RAG | Light-Medium | Complements Ollama |
| 3 | **Stable Diffusion (A1111)** | AI image generation — text-to-image, LoRA, ControlNet | Heavy | 8GB VRAM ideal for SDXL |
| 4 | **Frigate NVR** | Network video recorder + real-time object detection (people/vehicles) | Heavy | GPU for video decode + detection |
| 5 | **Whisper (faster-whisper)** | GPU-accelerated speech-to-text / subtitle generation | Medium | Transcribe media content |

### Homelab Utility
| # | Service | What it does | Footprint | Fit |
|---|---------|-------------|-----------|-----|
| 6 | **AdGuard Home** | Network-wide DNS ad/tracker blocking | Light (~100MB) | DNS-level ad block for all devices |
| 7 | **Gitea** | Self-hosted Git service (GitHub alternative) | Light | Manage Docker Compose configs |
| 8 | **n8n** | Visual workflow automation (400+ integrations) | Medium | Automates Jellyfin/Immich/qBittorrent |
| 9 | **Home Assistant** | Smart home automation platform | Light-Medium | Complements homelab |
| 10 | **SearXNG** | Privacy-respecting metasearch engine | Light | Private web search |

### Media Stack Extensions
| # | Service | What it does | Footprint | Fit |
|---|---------|-------------|-----------|-----|
| 11 | **Nextcloud** | Self-hosted cloud storage + collaboration | Medium-Heavy | 2.8TB exFAT for file sync |
| 12 | **Mealie** | Recipe manager + meal planning | Light | Simple Docker deploy |
| 13 | **BookStack** | Wiki/knowledge base (shelves → pages) | Light-Medium | Document homelab configs |

### Organization
| # | Service | What it does | Footprint | Fit |
|---|---------|-------------|-----------|-----|
| 14 | **HomeBox** | Home inventory + asset tracking | Light | Track homelab equipment |
| 15 | **Navidrome** | Music streaming server (Subsonic-compatible) | Light | Stream music collection |

**All have Docker Compose support. None conflict with existing stack.**

---

## Known Issues / Debugging Notes
- **exFAT storage:** `/mnt/storage` is exFAT — cannot rename folders with special chars (`:` `|`). Applies to all media on this mount.
- **SSH user is `r-server@`**, not `aarz@` — `aarz@` has password auth disabled.
- **Kuma API auth:** Direct SQLite query via SSH is the reliable method.
- **qBittorrent v5 no permanent password API:** Temp password regenerates on restart.

---

## Git History
| Commit | Description |
|--------|-------------|
| `fc18bfc` | New /mini endpoint for Dashy System Status tile |
| `d44b647` | fix: fully purge music data from r-server |
| `6bd5e1f` | refactor: split docker-compose into 4 functional groups |

---

## Remaining Issues
- [x] **Port 2283 ghost binding** — RESOLVED. Immich now properly on 2283 (container restarted cleanly).
- [ ] **exFAT storage** — `/mnt/storage` is exFAT, can't rename folders with special chars.

---

## Mission Repo
https://github.com/Aarz-aaryan/r-server
