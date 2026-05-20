---
date: 2026-05-19
title: Escaping GoPro cloud, migrating to Google Drive with gopro-upload
description: >-
  A Python CLI to leave GoPro Plus without clicking through the media library for
  days: chunked streaming to Drive, SQLite state, verify pass; survives sleep,
  Ctrl+C, and wrong CDN sizes.
tags:
  - python
  - tools
image: /images/post-6.jpg
---

**£4.99 a month** for GoPro Plus buys cloud storage. Bulk export is another matter. [gopro.com/media-library](https://gopro.com/media-library/) handles a clip; hundreds of gigabytes of action footage is slow, brittle, and feels built to keep you inside the product. I was paying to host video I could barely move.

[gopro-upload](https://github.com/andrewpatt24/gopro-upload) is a **Python 3.11+ CLI** that copies the GoPro cloud library to **Google Drive** with **chunked streaming**, **SQLite state**, and a **verify** pass. Chunks go straight to Drive; the full library never has to land on disk first.

## Why the web UI fails as migration

- Bulk export scales poorly past a handful of files.
- You do not own the platform; policy and throttle risk sit with GoPro.
- No official migrate-out API. Community tools such as [gopro-plus](https://github.com/itsankoff/gopro-plus) reverse-engineer `api.gopro.com` like the web app does.

I wanted a **Drive folder I control**, a job I could start overnight, interrupt with Ctrl+C, sleep the laptop, and resume without a new plan.

## Pipeline

1. **Inventory** via GoPro media search API (paginated).
2. **Stream** each asset in **~16 MB chunks** from the CDN into **resumable Drive upload**.
3. **Track** pending / in progress / done / failed in SQLite.
4. **Verify** GoPro, Drive, and DB agree at the end.

Peak disk: about **one chunk plus the DB** (tens of MB).

| Command | Purpose |
|--------|---------|
| `gopro-upload doctor` | Smoke-test GoPro + Drive |
| `gopro-upload inventory` | Metadata into SQLite |
| `gopro-upload migrate` | Upload pending (interruptible) |
| `gopro-upload verify` | Reconcile cloud, Drive, DB |
| `gopro-upload retry-failed` | Reset failed rows after auth/size fixes |

```bash
gopro-upload inventory
gopro-upload migrate --limit 1
gopro-upload migrate
gopro-upload verify
```

## Design

### Chunked streaming

Signed CDN URL per asset. **HTTP Range** reads feed Drive’s **resumable upload**. A 4 GB `.360` file does not need to sit on the SSD for the move.

### Resume

SQLite per-asset status; Drive keeps partial state where applicable. Stop, sleep, run `migrate` again; work continues from the remainder.

### `drive.file` scope

OAuth **`drive.file`**: the app only sees **files and folders it creates**. Run `gopro-upload init-folder` once; the CLI stores the folder id in `config.yaml`. A folder id copied from the Drive website will not work under this scope. Least privilege by intent.

### Pairing on verify

`appProperties.gopro_media_id` on each Drive file lets **verify** match cloud assets without filename guessing.

### GoPro auth via cookies

No public API key for “download my subscription library.” Session cookies (`gp_access_token`, `gp_user_id`) from DevTools or:

```bash
python extract_gopro_cookie.py @cookies.txt
eval "$(python extract_gopro_cookie.py @cookies.txt)"
```

**401** means refresh cookies from the browser.

## Sharp edges

**Inventory `file_size` vs CDN size.** Metadata can oversize the object; uploads stall with **HTTP 416**. The tool **probes CDN size** before transfer; `retry-failed` and `migrate` handle stragglers.

**Undocumented API.** Treat as a **migration utility**; endpoints can change.

**Secrets.** Do not commit `config.yaml`, `credentials.json`, cookies, or `data/migration.db`. README covers `~/.config/gopro-upload/` and revoking Google access.

## When to bother

Rare exports inside GoPro’s UI: you may never need this. Monthly fee plus a **second copy you control**, or cancelling Plus with the library still in the cloud: automation beats days in the media library.

Once my files sat in a Drive folder I owned, cancelling Plus stopped feeling like data loss.

## Code

[github.com/andrewpatt24/gopro-upload](https://github.com/andrewpatt24/gopro-upload) · MIT · Python 3.11+, GoPro cloud media, Google Cloud Drive API.

README: OAuth, cookies, then `doctor` → `inventory` → `migrate` → `verify`.

---

*Built out of annoyance; no affiliation with GoPro or Google. Respect their terms of service.*
