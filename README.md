# sentinel3-data-repo

Sentinel-3 OLCI ocean color for the US East Coast — chlorophyll-a, and the
harmful algal bloom indicators built on it — published as the map data
contract's grid files.

**Nothing runs yet.** This repository holds its founding plan and its rules;
there is no `pipeline/products.toml`, no cron, and nothing published. What
follows describes the intended shape, and says plainly where it is intent
rather than fact.

`PLAN.md` is the founding plan and running record — the upstream, the
measurements, and what is open. `DECISIONS.md` indexes the dated one-way
decisions. **Which document gets what, and what "update docs" means across
all eight repositories, is the doctrine block at the top of `CLAUDE.md`** —
the same text in all eight, held equal by the site's `check:docs`.

## What it will publish

Chlorophyll-a over the US East Coast, as a **multi-day merged composite**:

| | |
| --- | --- |
| source | NOAA CoastWatch, Copernicus Sentinel-3A **and** 3B OLCI, near-real-time |
| datasets | `noaacwS3AOLCIchlaSectorFIDaily`, `noaacwS3BOLCIchlaSectorFIDaily` |
| variable | `chlor_a`, mg m⁻³ |
| native resolution | 0.0025° ≈ **278 m** |
| extent | lat 29.81–45.19, lon −80.04 to −59.96 — Florida to Nova Scotia |
| upstream cadence | one frame per day per satellite, ~14:30–15:20Z |
| upstream latency | **2–4 days**, measured 2026-08-30 |
| upstream history | **90-day rolling window** — not an archive |

**The composite length is not decided.** Seven days is recommended and
measured (see below); the owner's call is open. The published extent and
resolution are likewise open, pending a byte measurement against the 1 GB
Pages cap.

## Why a composite, in one table

Coverage of everything ever observable, by composite length, measured
2026-08-30 across 36 frames and five weeks (both satellites, 0.15° sampling,
lat 30–45, lon −80 to −66):

| composite | S-3A only | S-3B only | **merged A+B** |
| --- | --- | --- | --- |
| 1 day | 11.8% | 5.9% | **17.3%** |
| 3 days | 31.6% | 18.1% | **43.5%** |
| 5 days | 46.6% | 30.3% | **62.5%** |
| **7 days** | 58.1% | 40.5% | **74.2%** |
| 10 days | 73.5% | 55.9% | **86.6%** |
| 14 days | 87.3% | 70.7% | **96.3%** |

Single-day coverage over a Chesapeake box across six consecutive days was
19.9%, 12.6%, **0.0%**, 7.4%, 7.2%, 0.5% — one day in six completely blank.
That is what makes the composite a requirement rather than a preference, and
merging the two satellites is worth roughly a doubling of the
single-satellite window.

The full method, and the caveat that late August is neither the cloudiest nor
the clearest season on this coast, are in `PLAN.md`.

## How it will run

The same arrangement as `espc-model-repo`, and for the same reasons:

- **No code here.** The orchestrator (`pipeline/orchestrate.py`) comes from
  `realtime-data-repo`; the fetchers and the published-file contract
  (`schema.ts`) come from `oceansensing.github.io`. Both are checked out at
  run time, so a change to either lands here on the next run rather than on
  any push here.
- **This repository carries `pipeline/products.toml`** — what it publishes,
  and nothing else executable.
- **Its own cron and its own fault domain.** A cloudy fortnight over the
  Atlantic must not hold back the currents, and HYCOM's outages must not hold
  back this.

Once there is a pipeline, this section gains the commands. There are none to
give yet, and inventing them here is how a README starts lying.

## Structure

```
PLAN.md         the founding plan and running record; measurements live here
CLAUDE.md       what must not be got wrong here, and the shared doc doctrine
DECISIONS.md    dated one-way decisions, D1 onward
pipeline/       products.toml — not written yet
```

## Licensing and attribution

The published data derives from **Copernicus Sentinel-3 OLCI**, processed by
EUMETSAT and served by **NOAA CoastWatch**. Any map drawing it owes that
attribution, and the site's attribution control is where it goes. The
upstream's own terms travel with the data: CoastWatch's ERDDAP states the
data may be used and redistributed freely but is not intended for legal use
and may contain inaccuracies.
