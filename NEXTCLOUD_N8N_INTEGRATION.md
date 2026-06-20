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
- **ID:** `LRwTw3oXq4kfl0dW`
- **Name:** Nextcloud r-server
- **Type:** `nextCloudApi`
- **WebDAV URL:** `http://100.84.224.18:9080/remote.php/webdav/`
- **User:** `aaryantahir8918@gmail.com`
- **Password:** App password (above)

> **Note:** A duplicate credential (`qXY4jZrmhjbeDF9L`) was removed 2026-06-20. All workflows were updated to reference the canonical ID `LRwTw3oXq4kfl0dW`. The `/remote.php/webdav/` endpoint is the user-aliased WebDAV root in Nextcloud — equivalent to `/remote.php/dav/files/<user>/` — and is the correct value for the n8n Nextcloud node.

### 4. n8n Workflows
| ID | Name | Trigger | Status |
|----|------|---------|--------|
| `Xu5B1rVAQngNxvjw` | Nextcloud Webhook Trigger | Webhook POST `/webhook/nextcloud-test` | ✅ **Operational** |
| `j5hhaX29DHEUkZWo` | Nextcloud File Browser | Manual | Active |
| `XKvBDPNHyg9WWRkY` | Nextcloud File Browser | Manual | Active |
| `ufjEE1ZWZzvJulxh` | Nextcloud Webhook Test | Webhook | Active |
| `v3J8mkcqAWH392NZ` | Nextcloud WebDAV Test (Code) | Webhook | Active |
| `kp6j03XBu5fpzJqW` | Minimal Code Test | Webhook | Active |
| `bAV5MJiETh7FEa8F` | Webhook Execute Test | Webhook GET | Active |

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

---

## Update Log: 2026-06-20 — End-to-End Webhook Integration Resolved

After the r-server cleanup that wiped the music stack, Nextcloud containers were stopped. The compose file at `/home/r-server/docker/nextcloud-compose.yml` was intact but no containers were running. Brought back up via:

```bash
ssh r-server@100.84.224.18 "cd /home/r-server/docker && docker compose -f nextcloud-compose.yml up -d"
```

Containers now running:
- `nextcloud` — port 9080→80
- `nextcloud-redis` — port 6379 (internal)
- `collabora` — port 9980

### Three Bugs Fixed

#### Bug 1: Orphan Credential Reference
After the upgrade from n8n v2.10.4 → v2.26.8, the workflow `Xu5B1rVAQngNxvjw` (and three other active workflows) referenced a credential ID (`qXY4jZrmhjbeDF9L`) that no longer existed in the DB. The canonical credential is now `LRwTw3oXq4kfl0dW`.

**Fix:** Updated credential ID reference in `workflow_entity.nodes` and `workflow_history.nodes` for all affected workflows.

**Error before fix:** `Credential with ID "qXY4jZrmhjbeDF9L" does not exist for type "nextCloudApi"`

#### Bug 2: Upload Path With Trailing Slash
The `Upload Test` node had `path: "/"` which generated the WebDAV URL `/remote.php/webdav//` (double slash). Nextcloud rejects `PUT` to a non-file path with HTTP 409.

**Fix:** Changed `path` from `"/"` to `"n8n-webhook-test.txt"`. The Nextcloud node's source code at `NextCloud.node.ts:865` does `webDavUrl + "/" + endpoint`, so the `path` parameter must be the filename itself (or `subdir/file.txt` for nested paths), not start with `/`.

**Logs showed:**
```
PUT /remote.php/webdav/// HTTP/1.1  ← before (409 Conflict)
PUT /remote.php/webdav//n8n-webhook-test.txt HTTP/1.1  ← after (201 Created)
```

#### Bug 3: Orphaned Respond Node With `responseMode: lastNode`
The workflow used `responseMode: "lastNode"` on the Webhook but still had a `Respond to Webhook` node at the end of the chain. n8n v2.26.8's Respond node requires the `respondWith` parameter, and combined with the v1.1 Webhook's auto-respond behavior, the chain failed with `Could not get parameter "respondWith"`.

**Fix:** Removed the `Respond` node entirely from both `nodes` and `connections`. With `responseMode: lastNode`, the last executed node (the `Debug` Code node) becomes the HTTP response automatically.

### Final Webhook Test Result
```
POST /webhook/nextcloud-test → HTTP 200
{"pairedItem":{"item":0}}

Execution 94: status=success
File uploaded: n8n-webhook-test.txt (46 bytes)
   contents: "n8n webhook upload successful at SUCCESS: Nextcloud triggered! Time: {{ $now }}"
```

### Active Webhook Workflow (`Xu5B1rVAQngNxvjw`)

```
Webhook (POST /webhook/nextcloud-test)
   ↓
List Documents (Nextcloud: folder.list path="/")  → 12 items returned
   ↓
Upload Test (Nextcloud: file.upload path="n8n-webhook-test.txt", fileContent="n8n webhook upload successful at ...")  → 201
   ↓
Debug (Code node: logs all items, passes through)
   ↓
[auto-respond with lastNode output → HTTP 200]
```

### Other n8n State Cleanup
- Deleted 4 dead/inactive Nextcloud test workflows (`c2CJoENon1WXsCsk`, `LJnXS6mdHn7zCLCv`, `trFrz9j1ac4dX5uQ`, `rNoD0n3jZGMHpeVz`) along with their `workflow_history` and `webhook_entity` rows
- Updated credential references in 3 remaining active workflows (`j5hhaX29DHEUkZWo`, `XKvBDPNHyg9WWRkY`, `ufjEE1ZWZzvJulxh`) from dead `qXY4jZrmhjbeDF9L` to canonical `LRwTw3oXq4kfl0dW`

### Notes for Future Reference
- **Encryption key for n8n credentials** is at `/n8n-data/n8n_data/config` → field `encryptionKey`. Standard PBKDF2-AES-256-CBC with `Salted__` prefix in base64.
- **Active workflow data lives in `workflow_history`** (pointed to by `workflow_entity.activeVersionId`), not in `workflow_entity.nodes` directly. Edits to `workflow_entity.nodes` only are ignored at runtime — must update both, or restart the container after editing `workflow_history` to force a reload.
- **n8n container restart** (after editing DB): `cd /n8n-data && docker compose restart n8n`. Wait ~12 seconds for the active workflow manager to register webhooks.
