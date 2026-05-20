---
date: 2016-12-31
title: Building Beercrawl, pub crawls across London solved with maps
description: >-
  Flask and Google Maps: two London pubs, walking or cycling, N stops, shortest
  route on a map. Side project for group chats that stall on pins.
tags:
  - python
  - web
image: /images/posts/beercrawl-london.jpg
---

*Pub crawl Saturday: start, end, how many stops?* Someone opens Google Maps, drops pins, and twenty minutes later the walking route still looks wrong.

[**Beercrawl**](https://github.com/andrewpatt24/Beercrawl) (**BeerCrawlr** in the UI) takes start pub, end pub, one to four stops, and walking or cycling. It searches bars along the corridor and returns turn-by-turn directions.

## Algorithm

Line from A to B. Search bars in the corridor. Pick intermediate stops that **minimise total distance**. Optional **rating-first** route via `getTopPlacesRoute`.

Accounts and social features omitted on purpose: form, map, beer.

## `BeerCrawler`

1. Geocode start and end.  
2. Midpoint = search centre.  
3. Radius = [haversine](https://en.wikipedia.org/wiki/Haversine_formula) from centre to start.  
4. Places radar search (`bar`, `beer`).  
5. Place details; missing ratings → 0 for experiments.  
6. `itertools.combinations` over stop count; Directions with `optimize_waypoints=True`; keep minimum distance.

Flask **`/directions`** JSON. Front end: autocomplete, mode toggle, stop slider; jQuery + Maps JS.

### Wapping smoke test

- **Start:** Town of Ramsgate, Wapping High Street  
- **End:** Prospect of Whitby, Wapping Wall  
- **Waypoint:** Captain Kidd  

Sensible riverside line; Zone 3 detours mean something broke.

## Stack

| Layer | Choice |
|-------|--------|
| Backend | Python, Flask |
| Maps | `googlemaps` Geocoding, Places, Directions |
| Front end | HTML, jQuery, Maps JS |
| Geometry | Haversine in `utils.py`; combinations in `itertools` |

`app.py`, `utils/crawl_object.py`, `utils/utils.py`, one template. `api_key` gitignored; enable Maps, Places, Directions in Google Cloud.

## What I kept

Separate **candidate search** from **route scoring** so `TopXBeerCrawler` (rating-first) could sit beside distance-first logic.

London pubs often lack ratings; default 0 favours chains with volume reviews. A heavier version would mix rating, distance, and hours.

The Wapping demo still beats the README.

## Run

```bash
python app.py
```

Clone [andrewpatt24/Beercrawl](https://github.com/andrewpatt24/Beercrawl), add `api_key`, install deps, pick two pubs, **Give me beer!**

PRs welcome; an anti-Wetherspoons filter would be a popular fork.

---

*Drink responsibly. The optimiser minimises distance.*
