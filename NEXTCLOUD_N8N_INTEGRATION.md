# Nextcloud + n8n Integration

## Overview

Self-hosted Nextcloud (r-server) connected to locally-hosted n8n (device), enabling Nextcloud nodes to replace Google Sheets and Google Docs in n8n workflows.

**Status: OPERATIONAL** ✅

---

## Infrastructure

| Service | Host | Port | URL |
|---------|------|------|-----|
| Nextcloud | 100.84.224.18 (r-server) | 9080 | `http://100.84.224.18:9080` |
| Nextcloud WebDAV | 100.84.224.18 | 9080 | `http://100.84.224.18:9080/remote.php/webdav/` |
| n8n | 100.100.35.6 (device) | 5678 | `http://100.100.35.6:5678` |

**Both hosts on Tailscale VPN** — direct IP connectivity.

---

## Setup Log (2026-06-19)

### 1. Port Binding Fix
Nextcloud was bound to `127.0.0.1:9080` (localhost only). Changed to `100.84.224.18:9080` to accept Tailscale connections.

**File:** `/home/r-server/docker/nextcloud-compose.yml`

```yaml
# Before
ports:
  - "127.0.0.1:9080:80"

# After
ports:
  - "100.84.224.18:9080:80"
```

Applied via:
```bash
sshpass -p 'aarz1947' ssh r-server@100.84.224.18
cd /home/r-server/docker
sudo sed -i 's/127.0.0.1:9080/100.84.224.18:9080/' nextcloud-compose.yml
sudo docker compose -f nextcloud-compose.yml down
sudo docker compose -f nextcloud-compose.yml up -d
```

### 2. App Password
Generated via Nextcloud OCC command:
```bash
sudo docker exec nextcloud php /var/www/html/occ user:auth-tokens:add \
  aaryantahir8918@gmail.com --name 'n8n-integration' --password-from-env
# Set NC_PASS env var or use interactive mode
```

**App password:** `TReUc7ZOrbGPiVH49RxHiciQnowBMNMrQmXWOR71iYUraSTOotTUOKSByDVnJszs5gANt4Xa`  
(Stored on device: `~/.n8n_nextcloud_app_password.txt`, chmod 600)

### 3. n8n Credential
- **ID:** `qXY4jZrmhjbeDF9L`
- **Name:** Nextcloud r-server
- **Type:** `nextCloudApi`
- **WebDAV URL:** `http://100.84.224.18:9080/remote.php/webdav/`
- **User:** `aaryantahir8918@gmail.com`
- **Password:** App password (above)

### 4. n8n Workflows
| ID | Name | Trigger | Status |
|----|------|---------|--------|
| `j5hhaX29DHEUkZWo` | Nextcloud File Browser | Manual (needs UI) | Active |
| `trFrz9j1ac4dX5uQ` | Nextcloud Integration Test | Schedule */1 * * * * | Active (buggy) |

---

## Verified Operations (WebDAV)

All 5 Nextcloud operations tested and confirmed working from n8n container:

```
1. LIST /      → HTTP 207, 6 files found
2. CREATE DIR  → HTTP 201
3. UPLOAD      → HTTP 201
4. DOWNLOAD    → HTTP 200, correct content returned
5. DELETE      → HTTP 204
```

---

## Known Issues

### n8n v2.10.4 — Workflow Execution API Bug
The `POST /rest/workflows/{id}/run` endpoint returns `"Cannot read properties of undefined (reading 'id')"` for all workflows. This is a **known n8n v2 bug** introduced in v1.93.0+. It affects:
- `n8n execute --id` CLI command (Task Broker port 5679 conflict)
- `/rest/workflows/{id}/run` REST API
- Activation of schedule/webhook trigger workflows (n8n throws iteration error on startup)

**Workarounds:**
1. **UI Execute:** Open `http://100.100.35.6:5678` → open workflow → click "Execute workflow" manually
2. **Webhook:** For webhook-triggered workflows, the webhook is registered in DB but activation fails. Use UI toggle instead.
3. **Schedule:** Schedule triggers are stored but activation errors prevent execution. Use UI to activate.

### Webhook Node — "object is not iterable"
The Respond to Webhook node throws `"Cannot read properties of undefined (reading 'id')"` during integrated test mode. Production webhook execution works if workflow is activated via UI.

---

## Nextcloud Node Capabilities (n8n-nodes-base.nextCloud v1)

| Resource | Operations |
|----------|-----------|
| **File** | Download, Upload, Copy, Move, Delete, Share |
| **Folder** | List, Create, Copy, Move, Delete, Share |
| **User** | Invite, Create, Update, Delete |

### Replacing Google Sheets:
- Use **Nextcloud Tables app** (Airtable alternative, installed on r-server Nextcloud)
- Or store CSV/JSON in Nextcloud Files and use Code node to read/write via WebDAV
- Nextcloud Tables supports spreadsheet-like data with REST API access

### Replacing Google Docs:
- Use **Nextcloud Text** app (collaborative markdown notes)
- Use **Nextcloud Files** for document storage
- Use **Collabora Online** (integrated CODE server at port 9980) for full doc editing
- Store .docx/.xlsx files in Nextcloud and manipulate via WebDAV

---

## How to Use

### Connecting from n8n
1. Open n8n UI: `http://100.100.35.6:5678`
2. Create new workflow
3. Add **Nextcloud** node
4. Select credential: **"Nextcloud r-server"**
5. Configure operation (e.g., List folder, Upload file)

### WebDAV Direct Access
```
URL:      http://100.84.224.18:9080/remote.php/webdav/
Username: aaryantahir8918@gmail.com
Password: TReUc7ZOrbGPiVH49RxHiciQnowBMNMrQmXWOR71iYUraSTOotTUOKSByDVnJszs5gANt4Xa
```

### Example: List Documents Folder
```json
{
  "resource": "folder",
  "operation": "list",
  "path": "/Documents"
}
```

### Example: Upload File
```json
{
  "resource": "file",
  "operation": "upload",
  "fileName": "/output/report.csv",
  "fileContent": "= Dynamic data from n8n workflow"
}
```

---

## GitHub
Repo: https://github.com/Aarz-aaryan/r-server
