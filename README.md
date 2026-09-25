# Collections generated using AI

The collections was craeted using an AI agent.

# qBittorrent TV Renamer — Postman Collections

Two Postman collections that rename torrent folder and episode files via the qBittorrent Web API for use with Jellyfin.

---

## Files

| File | Purpose |
|------|---------|
| `qbt-part1-rename-folder.postman_collection.json` | Renames the root torrent folder to your show name |
| `qbt-part2-rename-files.postman_collection.json` | Renames every episode file inside the folder |

---

## Prerequisites

- qBittorrent with Web UI enabled
- Postman desktop app
- An active qBittorrent Web UI session open in your browser (Postman shares the session cookie automatically — no separate login needed)

---

## Setup

### 1. Import both collections

In Postman: **Import** → select both `.json` files.

### 2. Set collection variables

Each collection has its own variables tab. Edit them before running.

**Part 1 variables:**

| Variable | Description | Example |
|----------|-------------|---------|
| `qbtBaseUrl` | Base URL of the qBittorrent Web UI (no trailing slash) | `http://127.0.0.1:8080` |
| `torrentHash` | Hash of the torrent to rename — copy from the Web UI | `8e16f20466c52fc4...` |
| `showName` | New root folder name and episode name prefix | `Sousou no Frieren` |
| `mode` | `dry` to preview, `apply` to commit | `dry` |

**Part 2 variables** (same as Part 1, plus):

| Variable | Description | Example |
|----------|-------------|---------|
| `seasonFolder` | Season sub-folder created inside the root | `Season 01` |

> Variables prefixed with `Auto-populated` or `managed` are set by the scripts — leave them blank.

---

## Usage

### Recommended workflow

Always run in `dry` mode first to verify the plan, then switch to `apply`.

---

### Part 1 — Rename Folder

> Renames the torrent's root folder (e.g. `Sousou no Frieren`) to your chosen show name.

**Run with the Send button — no Collection Runner needed.**

1. Set `mode = dry`
2. Send **request 1** — detects the existing root folder and logs what would change
3. Review the console output
4. Set `mode = apply`
5. Send **request 1** again, then **request 2** — the folder is renamed

**What each request does:**

| # | Request | API call | Notes |
|---|---------|----------|-------|
| 1 | Get Torrent Files (detect root folder) | `GET /api/v2/torrents/files` | Detects root folder, stores in `oldRootFolder`. In dry mode stops here. |
| 2 | Rename Root Folder | `POST /api/v2/torrents/renameFolder` | Only runs in apply mode. |

---

### Part 2 — Rename Files

> Renames every episode file to the format `Show Name S01E01.mkv` inside a season sub-folder.
> Run **after** Part 1 has completed.

**Must be run via the Collection Runner** — request 4 uses `pm.execution.setNextRequest()` to loop through every file automatically. Using the Send button only renames one file at a time.

1. Set `mode = dry`
2. Open the **Collection Runner**, select this collection, click **Run**
3. Request 3 fetches the file list, builds the rename plan, and prints it to the console
4. In dry mode the runner stops after request 3 — review the plan
5. Set `mode = apply`, run the Collection Runner again
6. Request 4 loops automatically until all episodes are renamed

**What each request does:**

| # | Request | API call | Notes |
|---|---------|----------|-------|
| 3 | Get Torrent Files (build rename queue) | `GET /api/v2/torrents/files` | Builds `fileRenameQueue`. In dry mode stops here and prints full plan. |
| 4 | Rename Episode File (loop) | `POST /api/v2/torrents/renameFile` | Loops via `setNextRequest` until queue is exhausted. Apply mode only. |

---

## File naming convention

Episodes are renamed using the `SxxExx` pattern found in the original filename.

```
{showName} S{season}E{episode}.{ext}
```

**Example:**

```
Frieren.Beyond.Journeys.End.AAC.WEBRip.TV.GroupName/Frieren.Beyond.Journeys.End.S01E04.Other.Peoples.Homes.1080p.mkv
  →  Sousou no Frieren/Season 01/Sousou no Frieren S01E04.mkv
```

**Skipped automatically:**
- `.nfo` sidecar files
- Any file with no `SxxExx` pattern in its name (logged to console)

---

## Authentication

Postman uses the `SID` session cookie from your active qBittorrent Web UI browser session. No username or password configuration is needed **as long as you have the Web UI open in your browser**.

If you get `403` errors, refresh the Web UI login page in your browser to renew the session cookie, then retry.

---

## Dry vs Apply mode reference

| `mode` value | Part 1 behaviour | Part 2 behaviour |
|---|---|---|
| `dry` (default) | Logs folder rename plan, **stops before request 2** | Logs full file rename plan, **stops before request 4** |
| `apply` | Executes folder rename | Loops through and renames all episode files |
