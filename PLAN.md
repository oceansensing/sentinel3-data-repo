# sentinel3-data-repo — the founding plan and running record

Ocean color from Sentinel-3 OLCI, for the US East Coast, and the harmful
algal bloom indicators that can be built on it. Built and publishing since
2026-08-31; this document is the feasibility work, its measurements, the
decisions they support, and the running record. Started 2026-08-30.

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

~~**The retrieval itself is sound inside the estuary**~~ — **CORRECTED
2026-09-02: sound in the MAIN STEM, empty in the tributaries.** The 0.015°
sample below is dominated by the main stem and the mouth; measured at native
resolution by region, the James, York and Potomac carry 1–5% of their pixels
on a day the mouth carries 76%. The measurement is section 4 below. The
original paragraph follows as written, because its numbers are real and its
conclusion was wrong at the granularity that matters for a HAB map. It was the
other worry — the standard OC4ME algorithm is known to struggle in turbid Case-2
water. On 08-23, at 0.015° sampling:

| where | valid | median chl |
| --- | --- | --- |
| inside the Bay (37.2–39.0 N) | 40.0% | 7.34 mg m⁻³ |
| Bay mouth / plume | 67.2% | 1.64 |
| offshore shelf | 12.0% | 0.39 |

Plausible estuarine values, and — at 0.015° sampling — no sign of wholesale
masking. The gradient from bay to shelf is the one an oceanographer would
expect. **What the sampling could not see is that the masking is wholesale
within about 5 km of every shore**, which is where the tributaries are.

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

### 3. What the build costs, measured 2026-08-31

The measurement Open item 2 was waiting for, taken against the live upstream
on the day D6 was decided.

| | measured |
| --- | --- |
| native frame, whole sector | 6150 × 8030 = 49,384,500 cells |
| one native frame over the wire | **188.5 MB in 10.7 s** (~17 MB/s), no chunking |
| frames in a 7-day window | **9** — S-3A 7, S-3B 2 |
| upstream per build | ~1.7 GB, ~96 s of transfer |
| published, native 0.0025° | ~234 MB at 4.98 bytes/cell |
| published, 0.05° overview | 0.6 MB |
| coverage vs observable water | **76.8%** |
| per-cell freshest age | median 2 d, p90 4 d, max 6 d |

**The most consequential number is the 10.7 s.** The design nearly went to
0.005° on the assumption that a 188 MB request would time out and force a
chunked fetcher. It does not, and measuring it before writing the design is
the only reason native was affordable. **Memory, not transfer, is the
constraint that remains**: nine native frames stacked for a median is 1.8 GB
of float32, so the fetcher works in horizontal bands — a sixth of the sector
at a time holds ~300 MB.

**S-3B runs further behind than the founding record says.** Latency was
recorded as 2–4 days for the pair; on 2026-08-31 S-3A was at 2026-08-29
(2 days) and **S-3B at 2026-08-26 (5 days)**. That is why a 7-day window held
9 frames and not 14 — S-3B contributed 2. The merge is still worth what D4
measured, but a reader of that entry should not expect seven frames each.

### 4. The tributaries: the loss is upstream, and it is the shoreline

Measured 2026-09-02, after the owner asked why the Chesapeake's estuaries are
nearly empty on the map and whether a quality flag was hiding them.

**No flag of ours.** The fetcher refuses only values outside 0–500 mg m⁻³ and
takes the 7-day median. The live tiles, the live overview and a composite
rebuilt from the upstream frames for the same window agreed **100% cell for
cell** over the Bay (tiles `37_-76` and `38_-76`, finite masks and values).
The upstream dataset carries one variable and no flags, so there is nothing
here to relax.

**Per overpass, same sky, same day.** Sentinel-3A on 2026-08-23, the clearest
recent day over the Bay, by region at native 0.0025°:

| region | pixels with a value | median chl |
| --- | --- | --- |
| Bay mouth (lon −76.3 to −75.9, lat 36.9–37.3) | **76.3%** | 3.3 mg m⁻³ |
| main stem (lon −76.5 to −76.0, lat 37.0–39.3) | 43.8% | 6.9 |
| Potomac (lon −77.1 to −76.3, lat 37.9–38.4) | 14.4% | 35 |
| James (lon −77.0 to −76.4, lat 36.9–37.3) | **4.7%** | 95 |
| York (lon −76.9 to −76.4, lat 37.2–37.5) | **1.4%** | 113 |

*(The boxes include land; the ratios between them are the point.)* The
tributaries were not under a different cloud than the mouth beside them.

**It is a shoreline effect, and it is wide.** Take every pixel that carried a
value in any of 22 frames (both satellites, 2026-08-16 to 08-31) as the
observable water. On the clearest frame, S-3A 2026-08-20, coverage of that set
by distance from its edge:

| distance from the water's edge | seen |
| --- | --- |
| 1 px (278 m) | **3.1%** |
| 2 px | 6.9% |
| 3 px | 12.4% |
| 4–5 px | 19.5% |
| 6–8 px | 32.3% |
| 9–12 px | 41.2% |
| 13–20 px | 52.8% |
| 21–40 px | 73.4% |
| beyond 40 px (11 km) | **94.6%** |

A loss zone roughly 5 km wide is not a straylight or land-adjacency mask —
those take one or two kilometers. It has the shape of the retrieval failing
in turbid, CDOM-rich water: OC4Me is a Case-1 blue-to-green ratio algorithm,
the baseline atmospheric correction returns negative blue reflectance there,
and the Level-2 flags drop the pixel before CoastWatch ever grids it. **This
mechanism is inferred from the numbers, not read from the flags** — the flags
are not served, and CoastWatch's product page documents none. The medians
that survive in the rivers, 35–113 mg m⁻³ against 7 in the main stem, are
OC4Me at its ceiling, so even the retrieved estuarine values deserve
suspicion.

**What the composite recovers.** In the live composite (refTime
2026-08-29T15:13Z), 0.05° blocks holding at least one native value: main stem
91.3%, Potomac 70.6%, York 66.7%, James 56.2%. The observable channel in each
river is a few pixels wide, so the shore-hugging part never returns a value in
any frame, and the 0.05° overview — a stride-20 point sample — keeps only
11.1% of the James's cells and 6.5% of the York's. That only matters on wide
views, where the tile tier stands down and rivers are sub-pixel anyway; a
block aggregate instead of a point sample would fix it if it ever matters.

### 5. Alternatives for the estuaries, measured 2026-09-02

Measured where a measurement was possible the same day; named as unmeasured
where it was not.

**Copernicus Marine's global OLCI product is the one candidate that measured
better, and it is reachable with credentials this project already holds.**
`cmems_obs-oc_glo_bgc-plankton_nrt_l3-olci-300m_P1D`: the same instrument,
both satellites merged per day, processed by ACRI's Copernicus-GlobColour
chain with the **POLYMER** atmospheric correction and an **OC5** coastal
algorithm blended with CI-Hu offshore. Variables `CHL`, `CHL_uncertainty`,
`flags` (LAND only). Gridded at 1/180° ≈ **620 m**, not the 278 m of the
CoastWatch sector — "300 m" is the swath, not the grid. Newest day on
2026-09-02 was 2026-08-31, so latency is one to two days against CoastWatch's
two to five. A subset of lat 36.5–39.5, lon −77.5 to −74.5 for 13 days moved
19 MB through the toolbox in one call.

Same boxes, same days, this product against CoastWatch's per-satellite frames:

| day | region | Copernicus (A+B, 620 m) | CoastWatch OC4Me (278 m) |
| --- | --- | --- | --- |
| 08-25 | James | **15.9%** | S-3B 3.9% |
| 08-25 | York | **9.0%** | S-3B 4.2% |
| 08-25 | Potomac | 18.4% | S-3B 12.7% |
| 08-25 | main stem | **54.0%** | S-3B 23.6% |
| 08-30 | James | **9.3%** | S-3A 5.4% |
| 08-30 | York | **8.7%** | S-3A 3.4% |
| 08-30 | Potomac | 23.0% | S-3A 16.4% |
| 08-23 | James | 3.6% | S-3A 4.7% |
| 08-23 | main stem | 48.9% | S-3A 43.8% |
| 08-19 | James | 15.3% | S-3A 14.7% |

And its own shoreline profile, on its clearest day (2026-08-19, 1 px ≈ 620 m):
**65.9% seen at one pixel from the edge**, 80.4% at two, 86.6% at three to
four, 94.1% at five to eight, 99% beyond. Against 3% at 278 m and 7–12% at
0.6–0.8 km for OC4Me above. **The shore-loss zone is about one pixel wide
instead of about twenty.** What limits it in the rivers is the 620 m grid
itself — the York is three or four of its pixels wide — which is why the
river-box fractions are only modestly higher than CoastWatch's while the
shoreline profile is transformed. Its estuarine medians (25–79 mg m⁻³) are as
high as OC4Me's survivors; no in-situ check was made.

**Ruled out or not applicable, with the measurement that ruled them out:**

- **VIIRS 750 m on CoastWatch** — 97 daily sector chlorophyll datasets
  checked by axis extent; none contains 37.2 N, 76.5 W. The Bay has no VIIRS
  sector, and 4 km global VIIRS is not an estuary product.
- **CoastWatch's Chesapeake merged product** (OLCI RE10 + VIIRS OC3, the
  turbid-water one) — still ends 2025-05-27. The East Coast Node's current
  Chesapeake page lists OLCI OC4ME, VIIRS OC3 and MODIS OC3 via HTTP/FTP
  rolling archives; no red-edge product is named. Email, not a scraper.
- **CBIBS** — the API answers with the documented testing key: 6 buoys
  reporting at 2026-09-02T04:12Z, **none listing chlorophyll** among its
  variables. Not a chlorophyll source today.
- **Maryland DNR Eyes on the Bay** — 15-minute chlorophyll fluorescence at
  continuous-monitoring stations, some telemetered; web query tools only, no
  API found. **VECOS** (VIMS) — the same class of station in Virginia; its
  site did not render to a fetch, so its access methods are unverified.
- **Chesapeake Bay Program DataHub** — an API exists, for the monthly cruise
  stations; a baseline, not a real-time source.

**Unmeasured, and why:**

- **EUMETSAT Level-2 WFR** carries `CHL_NN`, the neural-net Case-2 chlorophyll,
  with the full WQSF flags, at the true 300 m and within hours of the pass.
  It is the only source with an estuary-tuned OLCI algorithm at native
  resolution. It costs a new credential pair (EUMETSAT Data Store, `eumdac`),
  half-gigabyte swath granules, and our own gridding and compositing — a
  fetcher of a different kind from anything here. Not measured because it
  needs credentials this project does not hold.
- **Sentinel-2 MSI / Landsat OLI** at 10–30 m with Case-2 processors
  (ACOLITE, C2RCC) would resolve every creek, at a revisit of two to five
  days and a processing cost that is a project of its own.
- **PACE OCI** — 1.2 km; coarser than the problem.

**What this recommends**, for the owner rather than decided: the Copernicus
Marine product as a **second chlorophyll origin** for the Bay — the
credentials exist, the toolbox is installed, the pixel is twice as coarse and
the shoreline is twenty times better. Whether that is a Bay inset under the
Sector FI product or a peer layer is the open question below.

### Five things that will bite a fetcher, not two

The founding record numbers two above and names a third in prose; two more
were paid for on 2026-08-31. All five are numbered here so none hides in a
sentence.

3. **ERDDAP 403s the default Python `urllib` user agent.** `curl` is fine;
   `requests` with an explicit UA is fine. The failure is a bare
   `HTTP Error 403: Forbidden` with no hint. *(Recorded 2026-08-30.)*
4. **`curl` glob-expands `[` and `]`.** A griddap query pasted into `curl`
   without `-g` or percent-encoding returns an empty body and **exit 0** — no
   error, nothing to read. It looks exactly like a dataset that has gone away.
5. **Subset by INDEX, not by value.** The advertised extent rounds: latitude
   runs to **45.18625**, not the 45.19 quoted above, and a request for
   `(45.19)` is refused with a message about the axis maximum. Index form —
   `[0:1:6149]` — cannot round wrong, and does not move when the upstream
   re-grids by a hair. The same class of trap as the sibling's off-grid region
   edge, which held a whole publish tree for an afternoon.

---

## The decisions this supports

**Four of these landed as `DECISIONS.md` D1–D4 on 2026-08-30** — its own
repository, a composite rather than single overpasses, Sector FI whole, and
merging 3A with 3B. What follows is the reasoning behind them plus the part
that is still only a recommendation. **The numbering below is deliberately
NOT D-anything**, so it cannot be mistaken for the decision index.

### Recommendation 1: seven days for the composite window (landed as D5)

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

### Why Sector FI whole (landed as D3)

Florida to Nova Scotia, in one upstream read, because that is how the dataset
is cut and subsetting it would save nothing upstream. Whether the *published*
product is clipped tighter is a storage question, below, not an upstream one.

### Recommendation 2: which HAB layer to build first (open, for the owner)

Three candidates, in increasing order of how much this repository would have
to invent:

1. **Chlorophyll anomaly** — current composite against a rolling baseline.
   This is what NOAA's own HAB branch uses operationally (their anomaly is
   against a two-month mean), and it is derivable from data this repository
   will already hold. **Cheapest, and the one to build first.**
   CoastWatch also publishes a Chesapeake 7-day climatology,
   `noaacwecnOLCImultisensorCHLeastcoast7DayClim`, as a ready baseline.
2. **CyAN Cyanobacteria Index** — EPA/NASA/NOAA/USGS, OLCI-derived, for lakes
   and estuaries. The Sentinel-3 phase concluded in 2024 in favor of
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

- ~~**Storage.**~~ CLOSED 2026-08-31 as D6: measured at 367 tiles, 233 MB,
  native 0.0025° over the whole sector — see "What the build costs" below.
  The 1 GB Pages cap remains the constraint that shaped the ESPC split.
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
- **Whether the estuaries get a second origin.** Measured 2026-09-02: the
  tributaries are 1–5% covered per overpass upstream, by a shoreline loss zone
  about 5 km wide, and Copernicus Marine's OLCI product narrows that zone to
  about one 620 m pixel (sections 4 and 5). Whether to add it, and as what
  shape, is Open item 6.
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
  `DOC-DOCTRINE v1` block held byte-equal across all **ten** by the site's
  `check:docs` (written 2026-08-30). Adding it moved the doctrine's own count
  from four to five, and the site's sibling list with it; the currents/fields
  split later that day took it to eight.
- **~~It is the second repository of five to carry a `DECISIONS.md`~~ —
  CORRECTED 2026-08-31: all ten carry one.** When this was written only
  `oceanlet.js` and this repository had one, and it called that a gap in three
  others. The gap closed the next day, and `check:docs` gained a rule
  requiring a `DECISIONS.md` **tracked in git** in every one of the ten — so
  it is no longer a gap to note but a gate to keep green.

---

## Open

1. ~~**Composite length**~~ — **CLOSED 2026-08-31 as D5**: seven days.
2. ~~**Published extent and resolution**~~ — **CLOSED 2026-08-31 as D6**:
   Sector FI whole at native 0.0025°, tiles under a 0.05° overview, values
   linear at two decimals. The byte measurement it waited for is in
   "What the build costs".
3. **Which HAB indicator** — D-3 above; anomaly recommended first.
4. **Whether to pursue the node's `chl_switch`** — an email, not a scraper.
5. **Archive or not** — the upstream keeps 90 days and this repository would
   need its own history for any seasonal claim.
6. **The estuaries** — Copernicus Marine's `cmems_obs-oc_glo_bgc-plankton_nrt_l3-olci-300m_P1D`
   as a second origin for the Bay, inset or peer; measured 2026-09-02 in
   section 5, not decided.

## Method note

The coverage experiment is reproducible: 36 frames, both satellites,
0.15° sampling over lat 30–45 / lon −80 to −66, coverage measured against the
union of everything observed in the window. Re-running it in a different
season will give different numbers — **late August is not the cloudiest time
of year on this coast, and it is not the clearest either.** A winter re-run
is the honest check on the 7-day recommendation, and nothing here has done
it.
