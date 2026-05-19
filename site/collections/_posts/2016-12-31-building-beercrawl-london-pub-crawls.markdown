---
date: 2016-12-31
title: Building Beercrawl — pub crawls across London, solved with maps
description: >-
  A Flask app that finds pubs between two London landmarks and plots the shortest walking or cycling route — because planning a pub crawl should be fun, not a spreadsheet.
tags:
  - python
  - web
image: /images/post-2.jpg
---

Every group chat has the same problem: *“Pub crawl Saturday — where do we start, where do we end, and how many stops?”* Someone volunteers to open Google Maps, drops pins at random, and twenty minutes later you still do not have a route that actually makes sense on foot.

I built [**Beercrawl**](https://github.com/andrewpatt24/Beercrawl) (branded **BeerCrawlr** in the UI) to automate that bit of London logistics. You pick a start pub, an end pub, how many stops you want in between, and whether you are walking or cycling. The app finds bars along the corridor between those two points and hands you back turn-by-turn directions on a map.

## The idea in one sentence

Draw a line between pub A and pub B, search for bars in the corridor between them, then find the combination of stops that keeps the total journey as short as possible.

That is it. No accounts, no social features — just “give me beer” and a map.

## How it works

Behind the scenes, a `BeerCrawler` class orchestrates the Google Maps APIs:

1. **Geocode** your start and end addresses into latitude and longitude.
2. **Find the midpoint** between them — the centre of your search area.
3. **Calculate a radius** using the [haversine formula](https://en.wikipedia.org/wiki/Haversine_formula) from the centre to your start point, so the search bubble covers the ground you are actually willing to walk.
4. **Radar-search** for nearby places with keywords like `bar` and `beer` via the Google Places API.
5. **Enrich each result** with full place details (including Google ratings; missing ratings default to zero for later ranking experiments).
6. **Try combinations** of candidate stops for the number of pubs you asked for, request directions with `optimize_waypoints=True`, and keep the route with the **minimum total distance**.

There is also a rating-first variant (`getTopPlacesRoute`) that simply picks the highest-rated pubs in the search bubble — useful when you care more about reviews than metres walked.

The Flask app exposes this through a single JSON endpoint (`/directions`). The front end is a simple form: autocomplete for start and end, a travel-mode toggle (walking or cycling), and a slider for one to four stops. jQuery calls the API; the Google Maps JavaScript API renders the result.

There is a baked-in test case I used while developing — a classic Wapping crawl:

- **Start:** Town of Ramsgate, Wapping High Street  
- **End:** The Prospect of Whitby, Wapping Wall  
- **Expected waypoint:** Captain Kidd  

If the algorithm is behaving, you get a sensible riverside route with a proper pub in the middle, not a detour to Zone 3.

## Tech stack

| Layer | Choice |
|-------|--------|
| Backend | Python, Flask |
| Maps & places | `googlemaps` client — Geocoding, Places (radar + details), Directions |
| Front end | HTML, jQuery, Google Maps JavaScript API (Places Autocomplete + DirectionsRenderer) |
| Geometry | Haversine distance in `utils.py`; waypoint combinations via `itertools.combinations` |

The repo is small on purpose: `app.py`, a `BeerCrawler` in `utils/crawl_object.py`, helpers in `utils/utils.py`, and one main template. An `api_key` file holds your Google API key locally and is gitignored — you need Maps, Places, and Directions enabled in Google Cloud.

## What I learned

- **API design matters even for side projects.** Separating “find candidates” from “score routes” made it easy to swap strategies. There is an alternate `TopXBeerCrawler` that ranks purely by rating instead of minimising distance — handy when you care more about reviews than metres.
- **London breaks naive assumptions.** Ratings are sparse on smaller pubs; defaulting missing ratings to zero avoids crashes but nudges the algorithm toward chains with hundreds of reviews. A production version might blend rating, distance, and opening hours.
- **The fun part is the demo route.** Nothing sells the app like running the Wapping example and watching Captain Kidd appear as a waypoint.

## Try it yourself

Clone the repo, add your Google API key to `api_key`, install dependencies, and run:

```bash
python app.py
```

Open the app, enter two pubs you actually want to visit, hit **Give me beer!**, and see whether the map agrees with your mates’ instincts.

The code lives on GitHub: [andrewpatt24/Beercrawl](https://github.com/andrewpatt24/Beercrawl). Pull requests welcome — especially if you have ideas for “avoid Wetherspoons” as a filter.

---

*Drink responsibly. Walk home or get a licensed taxi. The algorithm optimises distance, not blood alcohol content.*
