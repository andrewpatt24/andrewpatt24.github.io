---
date: 2026-05-19
title: "My AI Golf Coach: turning three golf apps into one improvement loop"
description: >-
  Range data, on-course exports, and a course-management framework in one SQLite
  stack and local dashboard; practice and scorecard share one thread instead of
  three silos.
tags:
  - python
  - ai
image: /images/post-7.jpg
---

Tuesday on the range: **Rapsodo** for carry, dispersion, path versus face. Sunday on the course: **Garmin** for score, strokes-gained snippets, shot traces when the export cooperates. In between I was playing the [**Scoring Method**](https://thescoringmethod.com/) way (ESZ, DSZ, the 100-yard ring). Three apps, zero shared answer to *what should I work on this week?*

[**golf-analysis**](https://github.com/andrewpatt24/golf-analysis) ties range delivery to where strokes leak on the card. The name says AI golf coach; the implementation is structural: ingest, normalise, rank leaks, point practice. Swing-tip LLM optional.

## Three sources, one thread

| Source | Delivers | Gap |
|--------|----------|-----|
| **Rapsodo** | Carry, dispersion, club delivery | Link to on-course outcomes |
| **Garmin** | Score, SG samples, shot traces | Link to deliberate course strategy |
| **Scoring Method** | ESZ / DSZ intent | Persistence, trends, range priorities |

The target sentence looks like: *ESZ misses on par 4s cluster around ~150 yards; here are the Rapsodo sessions with wide approach dispersion.*

## One library, small connectors

All data lands in **SQLite** (`data/library.db`). Raw files live under `data/raw/`; ingest **dedupes on SHA256** so re-imports are safe.

Each source implements **`Connector`**: `can_handle(path)` and `ingest(path) → IngestPayload`. The CLI walks `data/raw/`, picks a connector, writes rows. Connectors in, tables out.

### Rapsodo

- CSV exports (`rapsodo_session_12345.csv` and variants). Headers differ; the connector maps aliases into **`range_shots`** (carry, offline, spin, launch, club path, etc.).
- Optional **`golf-ingest rapsodo-sync`**: R-Cloud via bearer token and endpoints from DevTools (same SPA-shaped API pattern as other personal tools). Session metadata can enrich club labels from a local snapshot.

### Garmin

1. **FIT** (`.fit` or zipped Connect activities) via **`fitparse`**: rounds, hole summaries, track points when present.
2. **Golf Community JSON** (`golf-export.json`): scorecards in `details`, `shotDetails` with pin and end positions, last-10 SG blocks. Extra geometry may sit in `extra_json`.

```bash
uv run golf-ingest garmin-golf-sync --out ./data/raw/garmin/golf-export.json
uv run golf-ingest garmin-sync
uv run golf-ingest rapsodo-sync
uv run golf-ingest ingest
```

**Garth** for Garmin auth; Rapsodo token in `secrets.json`. Keep both out of git.

```bash
uv run golf-ingest --db data/library.db ingest
uv run golf-ingest --db data/library.db info
```

## Scoring Method on shot data

Two anchors from [the framework](https://thescoringmethod.com/):

- **ESZ:** reach the 100-yard scoring ring in regulation for your tier.
- **DSZ:** hole out in three strokes or fewer once inside.

The repo computes ESZ/DSZ from `shotDetails` when pins and shot ends exist. Order of preference: **haversine** to the pin before/after each shot; **Garmin remaining-distance** by shot id; **straight-hole heuristics** when coordinates are thin. Scorecard-only proxies appear when geometry is missing, with **UI caveats** so proxies are labelled as such.

Strategy shows up as a trend next to fairways and penalties instead of a PDF on the nightstand.

## Dashboard

**React** (Vite, MUI) on **FastAPI**. Dev: API **8000**, UI **5173**, `/api` proxied. Runs locally.

| Tab | Question | Inputs |
|-----|----------|--------|
| **Strategy** | ESZ/DSZ and course management | Garmin export, scoring tiles, rollups |
| **Performance** | Where SG and round stats leak | Rollups, last-10 SG, summaries |
| **Training** | Range delivery | `range_shots`, scatter, sessions |
| **Plans** | Next focus | WHERE → WHY → WHAT, training blocks |
| **Settings** | Filters, paths | `GOLF_GARMIN_JSON`, DB path |

| Variable | Role |
|----------|------|
| `GOLF_LIBRARY_DB` | SQLite path |
| `GOLF_GARMIN_JSON` | Community export |
| `GOLF_DASHBOARD_SETTINGS` | Year and limits |

```bash
uv run golf-ingest --db data/library.db dashboard-api --reload
cd dashboard && npm install && npm run dev
```

[http://localhost:5173](http://localhost:5173)

## WHERE, WHY, WHAT

1. **WHERE:** Garmin, ESZ/DSZ, SG samples rank stroke cost on the course.
2. **WHY:** Range cohorts and dispersion suggest mechanical causes.
3. **WHAT:** Plans plus CLI (`analysis-plan-report`, `range-shots-report`) set practice focus.

The analysis-plan doc defines the loop. **pytest** and the CLI guard the numbers. Roadmap items (fuller SG baselines, per-club Rapsodo windows) sit on modular connectors, a repository layer, and an API separate from the UI.

## Stack

| Layer | Choice |
|-------|--------|
| Core | Python 3.11+, `uv`, SQLite |
| Ingest | Connector registry, SHA256 dedupe, `golf-ingest` |
| Sync | Garth, R-Cloud HTTP |
| API | FastAPI `/api/v1` |
| UI | React, Vite, MUI, Recharts |
| Tests | pytest (connectors, ESZ/DSZ, API) |

## Run it

Clone [andrewpatt24/golf-analysis](https://github.com/andrewpatt24/golf-analysis), `uv sync`, drop exports under `data/raw/rapsodo/` and `data/raw/garmin/`, `ingest`, set `GOLF_GARMIN_JSON`, start API and dashboard. README: secrets, sync, `scoring-method.md`, on-course methodology, system spec.

PRs welcome on connector edge cases and ESZ/DSZ when Garmin changes export shapes.

---

*Personal project; no affiliation with Rapsodo, Garmin, or The Scoring Method.*
