---
# r-server — Resource Server Mission

## Mission
Homelab server for Aaryan Tahir's photo/video backup (Immich), file sync + office (Nextcloud + Collabora), password management (Vaultwarden), infrastructure monitoring.

---

## Server Info
- **Hostname:** resource-server
- **Tailscale IP:** 100.84.224.18
- **OS:** Ubuntu 24.04.1 LTS (Server)
- **SSH:** `sshpass -p 'aarz1947' ssh r-server@100.84.224.18`
- **Hardware:** DDR3 32GB RAM, i5 (4 cores), 5TB HDD + 500GB SSD, **RTX 2070 8GB**
- **Storage mount:** `/mnt/storage` (2.8TB, exFAT — Linux permissions not supported)
- **Docker compose (core):** `/home/r-server/docker-compose.yml` (homepage, vaultwarden, uptime-kuma, glances)
- **Nextcloud stack:** `/home/r-server/docker/nextcloud-compose.yml` (Nextcloud + Redis + Collabora)
- **Immich stack:** `/home/r-server/docker/immich/docker-compose.yml` (runs separately)
- **Docker socket:** `/var/run/docker.sock`

---

## Service Status (2026-06-20, post-cleanup)

| | Service | Container | Port | Status | Notes |
|-|---------|-----------|------|--------|-------|
| ✅ | Homepage | homepage | 8383 | UP | r-server dashboard |
| ✅ | Vaultwarden | vaultwarden | 8080 | UP | Password manager |
| ✅ | Uptime Kuma | uptime-kuma | 3001 | UP | Monitors all services |
| ✅ | Glances | glances | 61208 | UP | System stats (host pid/net) |
| ✅ | Nextcloud | nextcloud | 9080 | UP | File sync, Docs, Sheets, Calendar, Contacts |
| ✅ | Nextcloud Redis | nextcloud-redis | — | UP | Nextcloud caching |
| ✅ | Collabora Office | collabora | 9980 | UP | Self-hosted Google Docs/Sheets alternative |
| ✅ | Immich | immich | 2283 | UP | **Fresh install 2026-06-20** — login as aaryantahir8918@gmail.com / aarz1947 |
| ✅ | Immich Postgres | immich-postgres | — | UP | DB on SSD, fresh |
| ✅ | Immich Redis | immich-redis | — | UP | Valkey 9, fresh |
| ✅ | Immich ML | immich-machine-learning | — | UP | GPU-accelerated |

**11 containers. 11 UP. 6/6 Kuma monitors UP.**

---

## Kuma Monitors (2026-06-20)
- **6 monitors active, all UP:**
  - Immich (ID 3) → http://100.84.224.18:2283
  - Vaultwarden (ID 13) → http://100.84.224.18:8080
  - Nextcloud (ID 14) → http://100.84.224.18:9080/status.php
  - Collabora Office (ID 15) → http://100.84.224.18:9980
  - Homepage (ID 16) → http://100.84.224.18:8383
  - Glances (ID 17) → http://100.84.224.18:61208
- **2026-06-20 fix:** Nextcloud monitor `accepted_statuscodes_json` was malformed (`[200-299]` instead of `["200-299"]`) — caused silent JSON parse error → 30s timeout. Fixed and Kuma restarted.

---

## Port Access Map
```
2283   → Immich (photo/video backup) — phone app connects here
3001   → Uptime Kuma (monitoring dashboard)
61208  → Glances (system stats agent)
8080   → Vaultwarden (HTTP, Tailscale IP only)
8383   → Homepage (r-server dashboard)
9080   → Nextcloud (file sync + productivity suite)
9980   → Collabora Office (Docs/Sheets backend)
```

---

## Cleanup Log (2026-06-20)
Aaryan requested full r-server deep cleanup + fresh Immich install. Done:

**Containers (11 → 11):**
- Immich data WIPED on /mnt/storage/immich (upload, library, encoded-video, profile, thumbs, backups, model-cache)
- Immich DB WIPED (/home/r-server/docker/immich-db)
- Immich Redis WIPED (/home/r-server/docker/immich-redis)
- Immich moved from core compose → its own /home/r-server/docker/immich/docker-compose.yml
- Fresh Immich v2.7.5 deployed, admin user created via /api/auth/admin-sign-up

**Kuma fixes:**
- Nextcloud monitor URL: `/` → `/status.php` (returns 200 instead of 302)
- Nextcloud monitor `accepted_statuscodes_json` JSON parse bug fixed (was `[200-299]`, now `["200-299"]`)
- Kuma container restarted to pick up DB changes
- All 6 monitors: 6/6 UP

**Homepage fixes:**
- Glances widget URL `:9400` → `:61208` (was pointing to non-existent port — old Glances API port)
- services.yaml: 3 occurrences of 61208, 0 of 9400
- Homepage container restarted

**Host cleanup:**
- /home/r-server: removed 13 cruft files (2× lidarr dbs, 7× fix_*.sh, apply_fix.py, validate_yaml.py, lidarr_add_indexer.py, vaultwarden_import.csv, photo-backup/ symlink)
- /home/r-server: removed 8 empty default Ubuntu folders (Desktop, Documents, Downloads, Music, Pictures, Public, Templates, Videos)
- /home/r-server: removed 2 backup compose files (bak.20260620, deployed)
- /home/r-server: removed weird 0-byte `\K[^` file
- /tmp: removed 4 cruft files (img-rm.log, create_admin*.sql, inspect_immich.py)
- /etc/nginx: stopped + disabled nginx service (no sites enabled, no use)
- /etc/nginx/sites-enabled: empty (removed default symlink, vaultwarden broken proxy)
- /etc/nginx/sites-available: empty (removed default, immich old config)
- /home/r-server: removed 2 empty default Ubuntu folders (Desktop, Documents, etc.)
- CUPS: cups-daemon + cups-bsd + cups-client + cups-common + cups-filters + cups-filters-core-drivers PURGED
- :631 (CUPS) no longer listening
- :80 (nginx) no longer listening
- :8443 (nginx vaultwarden) no longer listening
- Tailscale IP `100.114.187.58` was Aaryan's phone, NOT a brute-force — no firewall action needed

**Not changed (already clean):**
- nextcloud-redis / nextcloud-data / nextcloud-apps / nextcloud-config (Docker volumes, in use)
- `docker_nextcloud_redis` / `docker_nextcloud_data` (named volumes, in use — compose project name was "docker")
- Immich data on /mnt/storage/immich (clean fresh state)
- Glances, Vaultwarden, Homepage, Uptime Kuma, Nextcloud, Collabora services

---

## Hardware Utilization
- **CPU:** i5-2500K, load ~0.1, ~27°C
- **RAM:** ~3GB used / 31GB available
- **GPU:** RTX 2070 8GB — Immich ML uses it
- **Storage:**
  - **SSD (/)** — 48GB used / 387GB free (11%)
  - **HDD (/mnt/storage)** — 71MB used / 2.8TB free (1%)

---

## Key Credentials
- **r-server local account:** `r-server` / `aarz1947`
- **Immich:** `aaryantahir8918@gmail.com` / `aarz1947` (fresh, 2026-06-20)
- **Vaultwarden:** `aaryantahir8918@gmail.com` / `aarz1947`
- **Nextcloud:** `aaryantahir8918@gmail.com` / `aarz1947`
- **Uptime Kuma:** no credentials set
- **Homepage:** no auth

---

## Phone Setup for Immich
1. Install Immich app (iOS App Store / Google Play)
2. Server URL: `http://100.84.224.18:2283`
3. Login: `aaryantahir8918@gmail.com` / `aarz1947`
4. Enable "Backup" — photos/videos will auto-upload to /mnt/storage/immich (3TB HDD)

---

## Known Issues
- **CMOS battery dead** — needs replacement (CR2032, ~$3). Currently every cold boot requires F1 → Esc → Enter. Workaround: use `shutdown -P now` (preserves standby power to CMOS).
- **HDD mount silent failure post-cold-boot** — systemd may not auto-mount /mnt/storage after power loss. Workaround: `sudo mount /mnt/storage` or check `systemctl status mnt-storage.mount`.
- **exFAT permissions locked** — kernel exfat driver forces uid=1000. Cannot chown/chmod. Use Docker named volumes or External Storage for anything needing Linux perms.
- **N8N on Hermes host (100.100.35.6)** can reach `100.84.224.18:9080` (Nextcloud) since 2026-06-20 binding changed to Tailscale IP.

---

## Mission Repo
https://github.com/Aarz-aaryan/r-server
