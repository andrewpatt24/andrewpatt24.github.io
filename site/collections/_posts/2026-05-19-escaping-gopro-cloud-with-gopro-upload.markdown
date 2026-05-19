---
date: 2026-05-19
title: Escaping GoPro cloud — migrating to Google Drive with gopro-upload
description: >-
  A Python CLI born from frustration with GoPro Plus — chunked streaming, resumable
  uploads, and SQLite state to move your media library off gopro.com.
tags:
  - python
  - tools
image: /images/post-6.jpg
---

I pay **£4.99 a month** for GoPro Plus so my footage lives in the GoPro cloud. Fair enough — until I actually need the files back. Downloading from [gopro.com/media-library](https://gopro.com/media-library/) is slow, brittle, and feels like it was designed to keep you locked in, not to help you leave. I was paying to **host** videos I could not **easily access**.

So I built [gopro-upload](https://github.com/andrewpatt24/gopro-upload): a small Python CLI that copies your GoPro media library to **Google Drive** with low disk use, resume across sessions, and a sanity-check pass at the end. This post is the story behind it and how it works.

## The frustration

GoPro’s cloud product promises backup and access from anywhere. In practice:

- **Bulk export is painful** — fine for one clip, awful for hundreds of GB of action footage.
- **You’re paying for storage you don’t control** — if GoPro changes the product, throttles downloads, or you want a second copy elsewhere, you’re at their mercy.
- **There is no official “migrate out” API** — the same gap that led community tools like [gopro-plus](https://github.com/itsankoff/gopro-plus) to reverse-engineer how the web app talks to `api.gopro.com`.

I wanted my library in **my** Drive, with a tool I could run overnight, interrupt with Ctrl+C, and pick up again after the laptop sleeps. No “download everything to disk first, then upload.”

## What gopro-upload does

[gopro-upload](https://github.com/andrewpatt24/gopro-upload) is a **Python 3.11+ CLI** that:

1. **Inventories** your cloud library via GoPro’s media search API (paginated).
2. **Streams** each file in **~16 MB chunks** from GoPro’s CDN straight into a **resumable Google Drive upload**.
3. **Tracks state in SQLite** — which assets are pending, in progress, done, or failed.
4. **Verifies** that GoPro, Drive, and the local database agree when you’re finished.

Peak local disk use is roughly **one chunk plus the database** (on the order of tens of MB), not the full size of your library.

| Command | Purpose |
|--------|---------|
| `gopro-upload doctor` | Smoke-test GoPro + Drive connectivity |
| `gopro-upload inventory` | Pull library metadata into SQLite |
| `gopro-upload migrate` | Upload pending files (safe to interrupt) |
| `gopro-upload verify` | Reconcile cloud, Drive, and DB |
| `gopro-upload retry-failed` | Reset failed rows after fixing auth or size issues |

Typical session:

```bash
gopro-upload inventory
gopro-upload migrate --limit 1    # smoke test one file
gopro-upload migrate              # full run; Ctrl+C is fine
gopro-upload verify
```

## Design choices that mattered

### Chunked streaming, not download-then-upload

Each asset gets a signed CDN URL from GoPro. The tool uses **HTTP Range** reads on that URL and feeds chunks into Drive’s **resumable upload** API. Your SSD does not need to hold a 4 GB `.360` file just to move it.

### Resume across sessions

SQLite records per-asset status. Drive keeps partial upload state where applicable. Close the terminal, sleep the Mac, run `migrate` again — it continues from what’s left.

### Narrow Google scope: `drive.file`

The default OAuth scope is **`drive.file`**: the app can only see **files and folders it creates**, not your entire Drive. That means you run `gopro-upload init-folder` once so the CLI creates the destination folder and stores its ID in `config.yaml`. Pasting a folder ID from the Drive website will not work with this scope — and that is intentional for least privilege.

### Matching files on both sides

Each uploaded Drive file gets `appProperties.gopro_media_id` so **verify** can pair cloud assets to Drive objects without guessing from filenames alone.

### GoPro auth: browser cookies (for now)

GoPro does not hand out a public API key for “download my subscription library.” The tool uses the same session cookies the website uses (`gp_access_token`, `gp_user_id`), extracted from DevTools or a small helper:

```bash
python extract_gopro_cookie.py @cookies.txt
eval "$(python extract_gopro_cookie.py @cookies.txt)"
```

Cookies expire; when you see **401** from GoPro, refresh them from the browser. Not elegant, but honest about how the cloud product actually works today.

## Sharp edges I hit (and fixed)

**Inventory size vs CDN size.** GoPro’s metadata sometimes reports a `file_size` larger than the real CDN object. Uploads could stall around 50% with **HTTP 416**. The tool now **probes CDN size** and corrects the inventory before transfer; `retry-failed` plus `migrate` picks up stragglers.

**Undocumented API.** This follows the same class of integration as other community tools. GoPro can change endpoints or auth tomorrow — treat it as a **migration utility**, not a supported product surface.

**Secrets.** Never commit `config.yaml`, `credentials.json`, cookie files, or `data/migration.db`. The README lists what lives under `~/.config/gopro-upload/` and how to revoke Google access when you are done.

## Is it worth it?

If you are happy inside GoPro’s ecosystem and rarely export, you might tolerate the web UI. If you are paying monthly and want **a real second copy** under your control — or you are cancelling Plus and need everything out first — automating the move beats clicking through the media library for days.

For me, the win was psychological as much as technical: **stop renting access to my own footage**. Once the library sat in a Drive folder I owned, cancelling the subscription felt obvious instead of scary.

## Get the code

- Repository: [github.com/andrewpatt24/gopro-upload](https://github.com/andrewpatt24/gopro-upload)  
- License: MIT  
- Requirements: Python 3.11+, GoPro subscription with cloud media, Google Cloud project with Drive API enabled  

Clone, follow the README for OAuth and cookies, run `doctor`, then `inventory` → `migrate` → `verify`. Issues and PRs welcome if GoPro’s backend shifts and we need to adapt.

---

*Built out of annoyance, not affiliation with GoPro or Google. Use responsibly and respect their terms of service.*
