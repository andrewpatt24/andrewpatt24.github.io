---
date: 2026-05-19
title: Building an AI golf coach — unifying Rapsodo, Garmin, and the Scoring Method
description: >-
  A personal analytics stack that ingests range and on-course data into one SQLite library,
  then surfaces Strategy, Performance, and Training in a local dashboard — so practice
  and course management tell one story.
tags:
  - python
  - ai
image: /images/post-7.jpg
---

I had the gadgets. I did not have a coach.

**Rapsodo** on the range told me carry, dispersion, and whether my path was fighting my face. **Garmin** on the course told me score, strokes gained samples, and (when the export cooperates) shot-by-shot geometry. Separately, I was trying to play golf with [**The Scoring Method**](https://thescoringmethod.com/) — ESZ and DSZ, the 100-yard ring, penalties as first-class failures — but I was not connecting any of it. Range Monday lived in one app. Saturday’s round lived in another. The mental scorecard lived in my head.

[**golf-analysis**](https://github.com/andrewpatt24/golf-analysis) is my attempt to build an **AI golf coach** in software: not a swing tip generator, but a system that **unifies** where I lose shots (Garmin + Scoring Method) with how I deliver the ball (Rapsodo), then lays it out in a coherent flow so I can actually improve instead of hoarding CSVs.

## The problem: three truths, zero narrative

| What I had | What it answered | What it did not do |
|------------|------------------|---------------------|
| Rapsodo LM | “How far and how straight is this club?” | Tell me if that pattern shows up when it matters on the course |
| Garmin Golf | “What did I score, and where did SG leak?” | Tie into a simple on-course strategy I was already trying to use |
| Scoring Method (ESZ / DSZ) | “Did I reach the scoring zone and get down in three?” | Persist, trend, or inform range priorities |

Each tool was fine on its own. Together they were **fragmented**: different exports, different UIs, no shared notion of “this week’s problem.” I wanted one place that could say: *you are failing ESZ on par 4s because approach dispersion from 150 yards is wide — here are the Rapsodo sessions that prove it.*

That is the north star for the repo.

## Unify first: one library, modular connectors

Everything lands in a local **SQLite** library (`data/library.db`). Raw files stay under `data/raw/` as an archive; ingest dedupes on file hash so re-importing the same export is safe.

The ingest layer is deliberately boring in a good way. Each source implements a small **`Connector`** contract: `can_handle(path)` and `ingest(path) → IngestPayload`. The CLI walks your raw folders, picks a connector, and writes normalized rows.

### Rapsodo — range and practice

- **CSV exports** from the launch monitor (session files like `rapsodo_session_12345.csv`). Headers vary; the connector maps aliases for carry, offline, spin, launch, club path, and the rest into canonical `range_shots` rows.
- **Cloud sync** (optional): `golf-ingest rapsodo-sync` pulls sessions from R-Cloud using a bearer token and endpoint config captured from DevTools — same class of “undocumented SPA API” honesty as other personal tools. Session list metadata can enrich club labels via a local snapshot.

Practice data becomes queryable: gapping, landing side, dispersion bands, club comparisons — the **how** layer.

### Garmin — on-course

Two paths, one community export:

1. **FIT files** (`.fit` or zipped activities from Garmin Connect). Parsed with `fitparse` into `golf_rounds`, hole summaries, and track points when present.
2. **Golf Community JSON** (`golf-export.json`). This is the rich one: scorecards in `details`, global `shotDetails` with pin and end locations, last-10 SG sample blocks. The connector fills rounds and holes in SQL and attaches per-round shot traces into `extra_json` when needed for geometry work.

Sync commands keep you out of manual export hell when you want them:

```bash
uv run golf-ingest garmin-golf-sync --out ./data/raw/garmin/golf-export.json
uv run golf-ingest garmin-sync          # recent activity FIT/ZIP downloads
uv run golf-ingest rapsodo-sync         # R-Cloud sessions → data/raw/rapsodo/
uv run golf-ingest ingest               # import everything under data/raw/
```

**Garth** handles authenticated Garmin Connect access; Rapsodo uses your own token in `secrets.json`. Neither belongs in git.

### Ingest in one line

```bash
uv run golf-ingest --db data/library.db ingest
uv run golf-ingest --db data/library.db info
```

That is the spine: **connectors in, normalized tables out.**

## Scoring Method on real shot data

The Scoring Method is a **course-management framework**, not a stat pack. Two ideas drive it:

- **ESZ (Enter the Scoring Zone):** reach the 100-yard scoring ring in regulation for your tier.
- **DSZ (Down in Three):** once inside, get the ball in the hole in three strokes or fewer.

I was already trying to play that way. What I lacked was **measurement** tied to Garmin’s traces.

The repo computes ESZ / DSZ from `shotDetails` when the export includes pin positions and shot end points. The pipeline is documented in the repo’s on-course methodology: prefer **geometry** (haversine to the pin before/after each shot), fall back to **Garmin’s remaining-distance fields** by shot id, then **straight-hole heuristics** when coordinates are thin. Scorecard-only proxies still appear when shot geometry is missing — with explicit caveats in the UI so you do not confuse a proxy with ground truth.

That bridges “simple strategy on the course” with “what actually happened Saturday.” Strategy is not a PDF on the nightstand; it is a trend line next to your fairway and penalty rates.

## The dashboard: one coherent flow

The **React** dashboard (Vite + MUI) talks to a local **FastAPI** service. Dev mode: API on port 8000, UI on 5173 with `/api` proxied. No cloud required — this is a coach for *my* bag, on *my* machine.

Tabs mirror how I think about improvement:

| Tab | Question | Main inputs |
|-----|----------|-------------|
| **Strategy** | Am I managing the course and hitting ESZ/DSZ goals? | Garmin export, scoring-method tiles, ESZ/DSZ rollups, scorecard trends |
| **Performance** | Where do strokes gained and round stats say I leak? | Garmin bundle: round rollups, last-10 SG samples, round summaries |
| **Training** | What does the range data say about delivery? | SQLite `range_shots`: clubs, scatter, sessions, shape proxies |
| **Plans** | What should I work on next? | Analysis-plan framing (WHERE → WHY → WHAT), training-block ideas |
| **Settings** | Year filter, limits, paths | `GOLF_GARMIN_JSON`, library DB location |

**Strategy** is the Scoring Method home: ESZ/DSZ percentages, penalty and blow-up proxies, Stableford when recorded, and multi-metric trend charts over your scorecards. **Performance** is the Garmin analytics view — SG samples, course windows, round-by-round charts. **Training** is Rapsodo — the mechanical evidence. **Plans** is where the loops close: diagnostics from the course, hypotheses from the range, a prioritized practice direction.

Environment variables keep paths explicit:

| Variable | Role |
|----------|------|
| `GOLF_LIBRARY_DB` | SQLite path (range + ingested rounds) |
| `GOLF_GARMIN_JSON` | Community export for Strategy / Performance |
| `GOLF_DASHBOARD_SETTINGS` | Year and UI limits via the API |

```bash
# Terminal 1
uv run golf-ingest --db data/library.db dashboard-api --reload

# Terminal 2
cd dashboard && npm install && npm run dev
```

Open [http://localhost:5173](http://localhost:5173). The coach is local by design.

## “AI golf coach” — what that means here

I am not claiming a foundation model reads your swing video. The **intelligence** is structural:

1. **WHERE** — Garmin + ESZ/DSZ + SG samples rank what costs strokes on the course.
2. **WHY** — Range cohorts and dispersion explain plausible mechanical causes.
3. **WHAT** — Plans and CLI reports (`analysis-plan-report`, `range-shots-report`) turn that into a practice focus instead of random range bucket time.

The repo’s analysis-plan doc spells out that loop explicitly. The dashboard is the human-facing version; pytest and the CLI keep the numbers honest.

Future hooks (documented, not all shipped) include fuller strokes-gained baselines, optimal Rapsodo windows per club, and richer training blocks — but the **architecture** is already aimed there: modular connectors, storage behind a repository layer, API independent of the UI so a hosted or agent-driven client could sit on the same JSON later.

## Tech stack (short)

| Layer | Choice |
|-------|--------|
| Core | Python 3.11+, `uv`, SQLite |
| Ingest | Connector registry, SHA256 dedupe, `golf-ingest` CLI |
| Sync | Garth (Garmin), R-Cloud HTTP (Rapsodo) |
| API | FastAPI `/api/v1` |
| UI | React, Vite, MUI, Recharts |
| Tests | pytest across connectors, ESZ/DSZ, API routes |

## Try it

Clone [andrewpatt24/golf-analysis](https://github.com/andrewpatt24/golf-analysis), run `uv sync`, drop exports under `data/raw/rapsodo/` and `data/raw/garmin/`, ingest, point `GOLF_GARMIN_JSON` at your community export, and start the API + dashboard. The README walks through secrets, sync commands, and the docs folder (`scoring-method.md`, on-course methodology, full system spec).

If you have lived the same fragmented setup — launch monitor on Tuesday, watch on Sunday, strategy in your notebook — this is the glue I wished existed. Pull requests welcome, especially on connector edge cases and ESZ/DSZ when Garmin changes export shapes.

---

*Personal project, not affiliated with Rapsodo, Garmin, or The Scoring Method. Play responsibly; the algorithm optimises understanding, not your handicap marker at the 19th.*
