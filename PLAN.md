# sentinel3-data-repo — the founding plan and running record

Ocean color from Sentinel-3 OLCI, for the US East Coast, and the harmful
algal bloom indicators that can be built on it. Nothing is built yet: this
document is the feasibility work, its measurements, and the decisions they
support. Started 2026-08-30.

**Read "What was measured" before trusting any number elsewhere in here.**
Every figure below was taken against the live upstream on 2026-08-30, and the
ones that decide the product's shape were taken deliberately rather than
quoted from a catalog.

---

## What this repository is for

The first consumer's map draws temperature, salinity, currents, wind, waves
and ice. It draws no **biology**. Chlorophyll-a is the one satellite product
that shows where the water is productive rather than merely warm or moving,
and it is the base layer every operational HAB indicator is built from.

The owner's ask, 2026-08-30: ocean color and remote sensing of HABs, for the
whole East Coast, as a composite, merging Sentinel-3A and 3B.

**Why its own repository.** The same argument the ESPC split made: one
upstream, one fault domain, one storage budget against the 1 GB Pages cap. A
cloudy fortnight over the Atlantic must not be able to hold back the
currents, and the currents' HYCOM outages must not hold back this.

---

## The upstream, verified 2026-08-30

**NOAA CoastWatch serves Sentinel-3 OLCI chlorophyll through ERDDAP
griddap** — the same interface the site's existing fetchers already speak.
That is the single most important finding: this product needs a new fetcher,
not a new kind of fetcher.

| | |
| --- | --- |
| datasets | `noaacwS3AOLCIchlaSectorFIDaily`, `noaacwS3BOLCIchlaSectorFIDaily` |
| what | Copernicus S-3A / S-3B OLCI, near-real-time, Sector FI, Level 3 |
| variable | `chlor_a`, mg m⁻³ |
| resolution | 0.0025° ≈ **278 m** |
| extent | lat **29.81–45.19**, lon **−80.04 to −59.96** — Florida to Nova Scotia |
| cadence | **one frame per day per satellite**, ~14:30–15:20Z |
| latency | S-3A latest `2026-08-28T14:50:53Z`, S-3B `2026-08-26T15:07:34Z` — **2 to 4 days** |
| history | **90-day rolling window** — this is not an archive |
| institution | NOAA CoastWatch (EUMETSAT/Copernicus processing) |

**Sector FI covers the entire requested region in one dataset.** Found by
asking ERDDAP's advanced search for datasets intersecting a Chesapeake box;
four answered, and FI is the 300 m pair. There is no need to stitch sectors
for the East Coast, which is what would otherwise have set the region's
shape.

### Two contract details that will bite a fetcher

Both were found by a request failing, not by reading documentation.

1. **The grid is 4-D: `time, altitude, latitude, longitude`.** A query
   omitting `altitude` does not fail cleanly — ERDDAP maps the constraints
   onto the wrong axes and returns a 404 complaining that a latitude is
   greater than the altitude maximum. Pass `[(0.0):1:(0.0)]`.
2. **Latitude DESCENDS** (45.186 first). A range must be given north-first.

A third, for whoever writes the fetcher: **ERDDAP 403s the default Python
`urllib` user agent.** `curl` is fine; `requests` with an explicit UA is
fine. The failure is a bare `HTTP Error 403: Forbidden` with no hint.

---

## What was measured

### 1. Single-day coverage is not a product

Fraction of a Chesapeake box (lat 36.9–39.4, lon −77.2 to −75.6, sampled at
0.02°) carrying a valid retrieval, S-3A, six consecutive days:

| 08-23 | 08-24 | 08-25 | 08-26 | 08-27 | 08-28 |
| --- | --- | --- | --- | --- | --- |
| 19.9% | 12.6% | **0.0%** | 7.4% | 7.2% | 0.5% |

*(The box includes land, so water-only rates are higher. The day-to-day
swing is the point, and it is cloud.)*

**One day in six was completely blank.** A single-scene layer would show a
reader an empty map a third of the time, and would show them nothing at all
on the day they most want to look — which is usually the day after a storm.
This is why CoastWatch publishes 7-day medians of its own, and it is the
measurement that makes the composite a requirement rather than a preference.

**The retrieval itself is sound inside the estuary**, which was the other
worry — the standard OC4ME algorithm is known to struggle in turbid Case-2
water. On 08-23, at 0.015° sampling:

| where | valid | median chl |
| --- | --- | --- |
| inside the Bay (37.2–39.0 N) | 40.0% | 7.34 mg m⁻³ |
| Bay mouth / plume | 67.2% | 1.64 |
| offshore shelf | 12.0% | 0.39 |

Plausible estuarine values, and no sign of wholesale masking. The gradient
from bay to shelf is the one an oceanographer would expect.

### 2. How many days a composite needs — the experiment

**36 frames** (18 per satellite, 2026-07-22 to 2026-08-28), sampled at 0.15°
over lat 30–45, lon −80 to −66: 9,494 cells, of which **6,357 (67%) were
observed at least once** across the whole window. That set — "water we can
ever see here" — is the denominator; the remainder is land or permanently
unseen, and counting it would flatter every number below.

Coverage of that set by an N-day composite, averaged over every N-day window
that fits:

| composite | S-3A only | S-3B only | **merged A+B** | merge gain | marginal per extra day |
| --- | --- | --- | --- | --- | --- |
| 1 day | 11.8% | 5.9% | **17.3%** | +5.5 pt | — |
| 2 days | 21.6% | 11.9% | **30.9%** | +9.3 pt | +13.6 |
| 3 days | 31.6% | 18.1% | **43.5%** | +11.9 pt | +12.6 |
| 4 days | 40.4% | 24.6% | **54.4%** | +14.0 pt | +10.9 |
| 5 days | 46.6% | 30.3% | **62.5%** | +15.8 pt | +8.1 |
| **7 days** | 58.1% | 40.5% | **74.2%** | **+16.1 pt** | +5.9 |
| 10 days | 73.5% | 55.9% | **86.6%** | +13.1 pt | +4.1 |
| 14 days | 87.3% | 70.7% | **96.3%** | +9.0 pt | +2.4 |

**Merging the two satellites is worth about a doubling of the single-satellite
window** — 7 days of S-3A alone (58.1%) is beaten by 4 days of the pair
(54.4% at four, 62.5% at five). The owner's instinct to merge is confirmed
with a number, and the merge pays most at exactly the window that is
otherwise attractive.

---

## The decisions this supports

### D-1 (proposed): the default layer is a 7-day merged composite

**7 days is where the marginal return falls off**, and it is also what
CoastWatch itself publishes for this region, which means a reader comparing
against the official product is comparing like with like.

The tradeoff being made, stated plainly because it is the whole design:

- **1–3 days** is fresher and mostly holes. 43.5% at three days means more
  than half the water has no answer.
- **7 days** gives three quarters of what is gettable, and the trailing edge
  of the window is a week old.
- **14 days** gives 96% and is not a bloom map. A HAB smeared over a
  fortnight tells a reader where algae were, not where they are, and this
  map's readers act on it.

**A composite must not lie about its own age.** Every cell in a 7-day
composite is from *some* day in that window, and a reader cannot tell which.
So the product publishes a **per-cell age** alongside the value — the same
argument the currency gate makes for the model products, at pixel
granularity. A bloom whose freshest pixel is six days old should be able to
say so.

**Median, not mean, and not latest.** The median is what survives an
undetected thin cloud or a sun-glint edge; a mean is dragged by exactly the
outliers that most often are not water.

### D-2 (proposed): the region is Sector FI, whole

Florida to Nova Scotia, in one upstream read, because that is how the dataset
is cut and subsetting it would save nothing upstream. Whether the *published*
product is clipped tighter is a storage question, below, not an upstream one.

### D-3 (open, for the owner): what the HAB layer actually is

Three candidates, in increasing order of how much this repository would have
to invent:

1. **Chlorophyll anomaly** — current composite against a rolling baseline.
   This is what NOAA's own HAB branch uses operationally (their anomaly is
   against a two-month mean), and it is derivable from data this repository
   will already hold. **Cheapest, and the one to build first.**
   CoastWatch also publishes a Chesapeake 7-day climatology,
   `noaacwecnOLCImultisensorCHLeastcoast7DayClim`, as a ready baseline.
2. **CyAN Cyanobacteria Index** — EPA/NASA/NOAA/USGS, OLCI-derived, for lakes
   and estuaries. The Sentinel-3 phase concluded in 2024 in favour of
   Sentinel-2 and the data continues to be published and reprocessed
   annually — so **treat its latency as unverified**; this record has not
   measured it.
3. **A species indicator** — NCCOS's CBEPS produces daily nowcasts and 3-day
   forecasts of *Karlodinium veneficum* for Chesapeake Bay. That is a MODEL,
   not a satellite retrieval, and would be a different product with a
   different fault domain. Noted so it is not confused with the above.

**The recommendation is (1) first**, and to treat (2) and (3) as separate
products with their own currency, rather than folding them into an "HAB
layer" that means three things.

---

## What is not yet known

Named rather than implied, because a gap that looks like an omission gets
filled twice.

- **Storage.** The Chesapeake box alone is ~2.2 M cells per frame at native
  resolution; Sector FI whole is far larger. Nothing here is measured yet,
  and the 1 GB Pages cap is the constraint that shaped the ESPC split. **The
  published tier's resolution and extent are a storage decision waiting on a
  byte measurement**, exactly as the currents' were.
- **The East Coast Node's own `chl_switch` product** — the one in the owner's
  screenshot (`sentinel-3.2026237.0825.1529C.b.L3.CB3...chl_switch`,
  which corresponds to the `2026-08-25T15:15:56Z` overpass). It is 300 m,
  Chesapeake-specific, and uses a **switching algorithm** for turbid water
  that the standard `chlor_a` does not. It is **not on ERDDAP**; it lives on
  the node's own file server, and `eastcoast.coastwatch.noaa.gov/data_access.php`
  404s. If that algorithm materially beats `chlor_a` inside the Bay, it is
  worth an email to the node rather than a scraper.
- **A stale dataset that advertises itself as current.** CoastWatch's
  `noaacwecnOLCImultisensorCHLeastcoast7Day` is titled "2016-present" and its
  `time_coverage_end` is **2025-05-27** — over a year behind. It is not
  proposed as a source, but it is the reason **the currency gate points at
  this upstream from day one**: a title is not a measurement, and this
  repository's first published product should already know how to say it is
  stale.
- **Whether the 90-day rolling window needs an archive.** The upstream keeps
  90 days. Anything this repository wants to say about a season, or any
  anomaly baseline built from its own history, has to be accumulated here
  rather than fetched.

---

## How it would fit what already exists

Recorded because it is the argument for the effort being small.

- **The site's map already draws this shape.** A chlorophyll field is a
  scalar grid — the same thing `ScalarFieldLayer` draws for SST and SSS,
  through the same `DataTileLattice` tier ladder and the same `schema.ts`
  grid contract. No engine work is anticipated.
- **The orchestrator is shared.** `pipeline/orchestrate.py` lives in
  `realtime-data-repo` and both data repositories run it, pointed at their
  own workspace through `PIPELINE_ROOT`. This repository would carry a
  `pipeline/products.toml` and nothing else executable.
- **Two rules inherited from the sibling**, both learned the hard way there:
  every `roots` entry must be one the site's `test-schema.mjs --roots`
  publishes, or the orchestrator exits 2 and stops the publish; and a product
  that leaves takes its files with it.
- **The doc doctrine applies, and this repository is inside it.** It carries
  `README.md`, `CLAUDE.md`, `PLAN.md` and `DECISIONS.md`, with the identical
  `DOC-DOCTRINE v1` block held byte-equal across all five by the site's
  `check:docs` (written 2026-08-30). Adding it moved the doctrine's own count
  from four to five, and the site's sibling list with it.
- **It is the second repository of five to carry a `DECISIONS.md`.** Only
  `oceanlet.js` had one; the site and both other data repositories do not,
  measured 2026-08-30. The doctrine says all of them should, so that is a gap
  in three repositories rather than a precedent to follow here.

---

## Open

1. **Composite length** — 7 days proposed and measured; the owner's call.
2. **Published extent and resolution** — waiting on a byte measurement.
3. **Which HAB indicator** — D-3 above; anomaly recommended first.
4. **Whether to pursue the node's `chl_switch`** — an email, not a scraper.
5. **Archive or not** — the upstream keeps 90 days and this repository would
   need its own history for any seasonal claim.

## Method note

The coverage experiment is reproducible: 36 frames, both satellites,
0.15° sampling over lat 30–45 / lon −80 to −66, coverage measured against the
union of everything observed in the window. Re-running it in a different
season will give different numbers — **late August is not the cloudiest time
of year on this coast, and it is not the clearest either.** A winter re-run
is the honest check on the 7-day recommendation, and nothing here has done
it.
