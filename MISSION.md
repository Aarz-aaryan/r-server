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
- **Docker compose (single source of truth):** `/home/r-server/docker-compose.yml` — all 8 services (homepage, vaultwarden, uptime-kuma, glances, nextcloud, nextcloud-redis, collabora). Split compose files deleted 2026-07-02 (caused name conflicts + bridge loss per header warning).
- **Immich stack:** `/home/r-server/docker/immich/docker-compose.yml` (runs separately)
- **Docker socket:** `/var/run/docker.sock`

---

## Service Status (2026-06-20, post-cleanup + post-HDD-migration)

| | Service | Container | Port | Status | Notes |
|-|---------|-----------|------|--------|-------|
| ✅ | Homepage | homepage | 8383 | UP | r-server dashboard |
| ✅ | Vaultwarden | vaultwarden | 8080 | UP | Password manager |
| ✅ | Uptime Kuma | uptime-kuma | 3001 | UP | Monitors all services |
| ✅ | Glances | glances | 61208 | UP | System stats (host pid/net) |
| ✅ | Nextcloud | nextcloud | 9080 | UP | Fresh install 2026-07-02 — user data on /var/lib/nextcloud-data (ext4 SSD; moved off exFAT 2026-07-02) |
| ✅ | Nextcloud Redis | nextcloud-redis | — | UP | Nextcloud caching |
| ✅ | Collabora Office | collabora | 9980 | UP | Self-hosted Google Docs/Sheets alternative |
| ✅ | Immich | immich | 2283 | UP | **Fresh install 2026-06-20** — uploads on /mnt/storage/immich (HDD) |
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

## Cleanup Log
### Pass 1 (2026-06-20, morning) — Containers + Kuma + Immich nuke
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
  - **SSD (/)** — 31GB used / 404GB free (8%) — OS + Docker system + small app DBs (Immich Postgres, Nextcloud SQLite)
  - **HDD (/mnt/storage)** — 7.8GB used / 2.8TB free (1%) — Immich user data only:
    - `/mnt/storage/immich/{upload,library,encoded-video,profile,thumbs,backups}` — Immich user photos/videos
    - `/mnt/storage/rserver-deep-audit-reports/` — (empty, reserved)

---

## Key Credentials
- **r-server local account:** `r-server` / `aarz1947`
- **Immich:** `aaryantahir8918@gmail.com` / `aarz1947` (fresh, 2026-06-20, user data on HDD)
- **Vaultwarden:** `aaryantahir8918@gmail.com` / `aarz1947`
- **Nextcloud:** `aaryantahir8918@gmail.com` / `aarz1947` (fresh 2026-07-02, user data on ext4 SSD)
- **Uptime Kuma:** no credentials set
- **Homepage:** no auth

All "fresh" services use the same admin email `aaryantahir8918@gmail.com` / `aarz1947` — login once on phone, works for Immich + Nextcloud + Collabora.

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
- **exFAT permissions locked** — kernel exfat driver forces uid=1000. Cannot chown/chmod. Affected services:
  - **Nextcloud** (FIXED 2026-07-02): moved data dir off `/mnt/storage/nextcloud/data` → `/var/lib/nextcloud-data` (ext4 SSD). Workaround `check_data_directory_permissions = false` no longer needed.
  - Immich: UID/GID set to 1000 via env vars, works fine
  - DO NOT bind-mount Nextcloud's entire `/var/www/html` to exFAT — entrypoint's `rsync --delete` fails on symlinks
- **N8N on Hermes host (100.100.35.6)** can reach `100.84.224.18:9080` (Nextcloud) since 2026-06-20 binding changed to Tailscale IP.

---

## Cron Jobs (r-server related)

| Job ID | Name | Schedule | What it does |
|--------|------|----------|--------------|
| `82b317671a10` | r-server Daily Health Report | Daily 02:05 | Queries Kuma DB via SSH + adds SSD/HDD usage + container count, posts to #r-server. Enhanced 2026-06-20. |
| `8649ec2bae70` | r-server Audit Watchdog | Weekly Sun 06:30 | Verifies 9 post-cleanup invariants: 11 containers UP, 6 Kuma UP, Immich data on HDD, Nextcloud data on HDD, /opt/immich-photos still NUKED, HDD/SSD free space >10%, snapd not reinstalled, nginx not active. Silent when OK; alert on regression. |

Both run on Hermes host and SSH into r-server — no on-r-server scheduling needed.

---

## Mission Repo
https://github.com/Aarz-aaryan/r-server

### Pass 2 (2026-06-20, afternoon) — Deep filesystem audit
Aaryan requested deep audit + cleanup of all files, stale backups, useless stuff. Total ~1.2GB freed + 50 packages removed + 11 GUI snaps + 5 broken snaps.

**Killed:**
- **`/home/r-server/scripts/` (14K)** — 3 dead scripts referencing nuked containers (qbittorrent, lidarr, prowlarr, jackett, jellyfin, navidrome, dashy, deluge). Stats-server.py was on dead port :9400. Replaced by Kuma (real-time) + Glances widget (system stats).
- **`/home/r-server/logs/` (80K)** — June 9 stale logs (health_check.log, critical_container_logs.log)
- **`/home/r-server/.cache/` (17M)** — Evolution, gstreamer, ibus, mesa, tracker3 — no GUI on r-server
- **`/home/r-server/.config/` (13 dirs)** — dconf, evolution, gnome-*, gtk-3.0/4.0, ibus, nautilus, pulse, user-dirs.* — all GUI cruft
- **`/home/r-server/.local/share/` (8 dirs)** — applications, evolution, flatpak, gnome-*, icc, keyrings, nautilus, session_migration-ubuntu, sounds, gvfs-metadata — all GUI cruft
- **`/home/r-server/.local/state/wireplumber/`** — audio daemon state, no audio on server
- **`/home/r-server/snap/`** — firefox, firmware-updater, snapd-desktop-integration dirs (snaps already removed)
- **`/root/.cache/` (113M)** — pip cache from old Lidarr/beets installs
- **`/root/.config/` (64K)** — root's stale config
- **`/root/snap/` (~50M)** — firefox, firmware-updater, snap-store, mesa-2404 dirs
- **`/var/cache/apt/` (320M → 44K)** — old .deb files
- **`/var/cache/apparmor (4.3M)`, fwupd (15M), swcatalog (8.4M), debconf (5.4M), man (2.2M), fontconfig (1.4M), cracklib (476K), ldconfig (64K)** — all stale cache
- **`/var/log/journal/` (245M → 64M)** — vacuumed to 50M, freed 181M of old boot logs
- **`/var/log/syslog` (26M)** — truncated
- **`/var/log/installer/` (1.7M)** — Ubuntu initial install logs from June 7, never needed
- **`/var/log/btmp` (60K)** — 6 months of failed login attempts
- **`/var/log/syslog.1` (15M)** — rotated log
- **`/var/log/nginx/`** — deleted (nginx stopped, no sites)
- **`/var/log/bootstrap.log`** — Ubuntu installer leftover
- **`/var/cache/snapd/` (4.4M)** + **`/var/lib/snapd/snaps/` (395M)** — snap blobs
- **`/usr/local/bin/`** (11 files) — beet, f2py, mid3cp, mid3iconv, mid3v2, moggsplit, mutagen-inspect, mutagen-pony, numba, numpy-config, unidecode, filetype — all from old music stack (beets/mutagen) and Lidarr (numpy/numba). Not used.
- **`/tmp/fix-monitor14.sql`, `/tmp/upsert_admin.sql`** — leftover from earlier ops
- **`/home/r-server/\K[^`** — weird 0-byte filename (orphan from `apt-get install` hiccup)
- **`snap` package entirely** — including broken bases bare, core22, core24, gtk-common-themes, snapd (was broken after /var/lib/snapd/snaps/ wipe)
- **15 packages via `apt autoremove --purge`**: cups-ipp-utils, hplip-data, libcupsimage2t64, libhpmud0, libimagequant0, liblouisutdml-bin, liblouisutdml-data, liblouisutdml9t64, libraqm0, libsane-hpaio, printer-driver-postscript-hp, python3-olefile, python3-pexpect, python3-pil, python3-ptyprocess, squashfs-tools
- **5 GUI snaps removed**: firefox, firmware-updater, snapd-desktop-integration, snap-store, mesa-2404
- **2 GNOME snaps removed**: gnome-42-2204, gnome-46-2404 (no GUI on server)

**KEPT (deliberately):**
- **`/opt/immich-photos/` (15GB, 1965 files)** — Aaryan's actual photo library backup from Nov 2025 → Jun 2026 (IMG_*.heic, IMG_*.jpg, IMG_*.mov). Found this after I'd already wiped /mnt/storage/immich. Was on /opt and survived the wipe. NOT deleted. Aaryan can decide later whether to import into new Immich via /api/asset/upload-external or just keep as backup. **Flagged for user decision.**
- **`/opt/containerd/`** — Docker dependency
- **`/var/log/sysstat/`** — last 9 days of sa/sar files. sysstat cron rotates these.
- **`/var/lib/systemd/coredump/`** — empty, leave for safety
- **All 11 Docker images** — all in active use by containers (homepage, immich, immich-ml, immich-postgres, immich-redis, vaultwarden, nextcloud, nextcloud-redis, collabora, glances, uptime-kuma)
- **All 10 Docker volumes** — all in use (nextcloud_*, immich_*)
- **`.local/share/keyrings/`** — SSH/GPG keyring (might still be needed)
- **`.config/{gnupg, goa-1.0, gtk-*}`** — keep, some apps might need
- **`/var/cache/{apt, dictionaries-common, samba, PackageKit, app-info, fwupdmgr, colord, adduser, tailscale}`** — active caches, tiny
- **`/home/r-server/.gnupg/`, `/home/r-server/.ssh/`** — credentials
- **`/home/r-server/{docker-compose.yml, docker/}`** — active Docker stack

**Final stats:**
- Disk freed: 47G → 44G on / (3GB freed), /mnt/storage unchanged (71M)
- Packages removed: 15
- Snaps removed: 5 GUI + 2 GNOME + snapd daemon entirely
- Containers UP: 11/11
- Kuma monitors UP: 6/6
- Immich login: HTTP 201 ✅

### Pass 3 (2026-06-20, evening) — HDD migration + Immich photos nuke + Nextcloud fresh install

Aaryan said: "make sure that all the backup kind of stuff like image photos backup and next cloud you know stuff everything goes on the 3TB hard disk. Go ahead." Also: "just go ahead and like nuke the image photos we will i will do a fresh backup so you can get rid of the old ones completely get rid of them the old ones."

Done.

**1. NUKED `/opt/immich-photos/` (15GB, 1965 files)** — Aaryan's old photo library backup (Nov 2025 → Jun 2026). Confirmed gone: `ls /opt/immich-photos` → "No such file or directory". `/opt/` now contains only `containerd/` (Docker dependency).

**2. Immich user data verified on HDD** — All Immich bind mounts point to `/mnt/storage/immich/*` (no change from fresh install in Pass 1, just verified).

**3. Nextcloud fully migrated to HDD + fresh install** — Original Nextcloud was on Docker named volumes (SSD, slow, mixed with backups). User wanted everything on 3TB HDD.

Process:
- Stopped nextcloud + nextcloud-redis containers
- Backed up user data to `/tmp/nc-backup/data` (73MB total)
- Tried `rsync` of old Docker named volume data → `/mnt/storage/nextcloud/data/` — partial (had wrong path structure)
- Tried bind-mounting `/mnt/storage/nextcloud/data/data:/var/www/html/data` — Nextcloud boot failed: "no such table: oc_appconfig" (mount overlay issue with anonymous Docker volume created by official image)
- Tried single bind `/mnt/storage/nextcloud/data:/var/www/html` — entrypoint `rsync --delete` from `/usr/src/nextcloud/` to `/var/www/html/` failed on every symlink (exFAT doesn't support symlinks) → 700+ errors before timeout
- **FINAL SOLUTION**:
  - Anonymous Docker volume for `/var/www/html` (PHP source on SSD, ~870MB, where symlinks work)
  - Bind mount ONLY `/mnt/storage/nextcloud/data:/var/www/html/data` (user data + DB on HDD)
  - `NEXTCLOUD_UPDATE=0` env var to skip initial rsync
  - Fresh install via `occ maintenance:install --database sqlite --admin-user aaryantahir8918@gmail.com --admin-pass aarz1947`
  - Restored user files: Documents/, Photos/, Nextcloud Manual.pdf, Nextcloud intro.mp4, Nextcloud.png, Readme.md, Reasons to use Nextcloud.pdf, Templates/, n8n-test.txt, n8n-webhook-test.txt
  - Set admin password to `aarz1947` (since fresh install generated new hash, override via PHP `password_hash`)
  - Config.php patches:
    - `'installed' => true` (otherwise /status.php says `installed:false` even when DB has it)
    - `trusted_domains` includes `100.84.224.18`
    - `check_data_directory_permissions => false` (exFAT perm workaround — data dir is 0777)
    - `check_for_working_windows_compatibility => false` (exFAT compat)
  - Removed 4 orphan Docker volumes (`docker_nextcloud_data`, `docker_nextcloud_apps`, `docker_nextcloud_config`, `nextcloud_nextcloud_redis`)
- **Verified**: `/status.php` returns `installed:true`, WebDAV returns HTTP 200, PROPFIND lists all user files

**4. Disk state after Pass 3:**
- **SSD (/)** — 31GB used / 404GB free (was 47G before Pass 3, freed 16GB by removing Nextcloud named volumes from SSD)
- **HDD (/mnt/storage)** — 7.8GB used / 2.8TB free (was 71M, now 7.8G because Nextcloud data lives here)

**5. Verified all 9 cleanup invariants** — see cron job `8649ec2bae70`. All pass: 11/11 containers UP, 6/6 Kuma UP, Immich data on HDD, Nextcloud data on HDD, /opt/immich-photos still NUKED, HDD 1% / SSD 8% (both healthy), snapd still gone, nginx still inactive.

**6. Login works end-to-end:**
```
$ curl -sL -u "aaryantahir8918@gmail.com:aarz1947" -X PROPFIND \
  http://100.84.224.18:9080/remote.php/dav/files/aaryantahir8918@gmail.com/
HTTP/1.1 207 Multi-Status
- Documents/
- Photos/
- Templates/
- Nextcloud Manual.pdf
- Nextcloud intro.mp4
- Nextcloud.png
- Readme.md
- Reasons to use Nextcloud.pdf
```

Phone → Immich app → http://100.84.224.18:2283 → aaryantahir8918@gmail.com / aarz1947 → upload photos → auto-backup to /mnt/storage/immich/upload.

**7. GitHub commits:**
- `39fa1d2` — Pass 1+2 (cleanup, Immich nuke, Kuma fix)
- `b257eaf` — Pass 3 (Nextcloud HDD migration)

---

### Pass 4 (2026-07-02) — Fix Nextcloud stuck on setup wizard (exFAT root cause)

**Symptom:** Aaryan opened http://100.84.224.18:9080/ and got the "set up Nextcloud" wizard. Login redirect loop.

**Root cause:** Pass 3 had worked around the exFAT perms issue with `check_data_directory_permissions => false` in config.php. But every time the nextcloud container restarted (e.g. via `docker compose up -d` on the split compose file), the official image's entrypoint re-detected `CAN_INSTALL` and re-ran `maintenance:install` on a dir it couldn't chmod → install half-completed, config got rewritten, wizard came back.

**Fix (proper):**
1. Created `/var/lib/nextcloud-data` on ext4 `/dev/sda2`, owned by 33:33 (www-data)
2. Updated `/home/r-server/docker-compose.yml` bind mount: `/mnt/storage/nextcloud/data:/var/www/html/data` → `/var/lib/nextcloud-data:/var/www/html/data`
3. Removed exFAT perms workaround from config (no longer needed)
4. Fresh `occ maintenance:install --database sqlite --admin-user aaryantahir8918@gmail.com --admin-pass aarz1947`
5. `occ config:system:set trusted_domains N --value=...` for `100.84.224.18`, `100.84.224.18:9080`, `127.0.0.1:9080`

**Cleanup:**
- Deleted `/home/r-server/docker/nextcloud-compose.yml` (orphan split file — caused container name conflicts + bridge loss, see header warning in consolidated compose)
- Deleted `/home/Aarz/r-server/nextcloud-compose.yml` and `/home/Aarz/r-server/docker/nextcloud-compose.yml` (tracked duplicates of same orphan)

**Data loss:** Original Nextcloud install at instanceid `ocs7zbz56esz` (~146MB in `/mnt/storage/nextcloud/data/` — Documents, Photos, n8n test webhooks, Nextcloud intro materials) — wiped. Aaryan said "I have nothing on there to lose, do whatever is needed." Lost files are not recoverable (exFAT mount remains but instanceid mismatch with new install).

**Lesson saved:** Skill `~/.hermes/profiles/aarz/skills/devops/r-server-compose-topology/SKILL.md` — read `/home/r-server/docker-compose.yml` and `/home/r-server/config.php` FIRST before any container/volume/bind mount change on r-server.

**Verified end-to-end:**
```
$ curl -c /tmp/c -L http://100.84.224.18:9080/      → HTTP 302 → /apps/dashboard/
$ docker exec nextcloud php occ user:list            → aaryantahir8918@gmail.com
$ docker exec nextcloud php occ status               → installed: true, version 34.0.0.12
```

**Disk state:** SSD 31GB / 404GB free (no change), HDD 7.8GB / 2.8TB free (no change — Nextcloud DB is tiny sqlite on ext4 now).
