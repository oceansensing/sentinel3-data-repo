# Decisions

Dated, irreversible-leaning decisions, one entry each, newest last. The
reasoning lives in `PLAN.md`; this file is the index of what was decided and
when, so a future reader never re-derives whether a door was walked through.

**What counts as one-way here** is worth stating, because a data repository's
doors are not an engine's. Three shapes: a decision that puts bytes in
readers' hands under a shape they will code against; a decision about which
repository owns a product, since moving one costs a migration in two places;
and a decision that forecloses an upstream. Tuning a threshold is none of
those.

## D1 — 2026-08-30 — Its own repository, not a product inside another

Sentinel-3 ocean color goes in `sentinel3-data-repo` rather than into
`realtime-data-repo` or `espc-model-repo`.

The same argument the ESPC split made and measured: **one upstream, one fault
domain, one storage budget** against the 1 GB Pages cap. A cloudy fortnight
over the Atlantic must not be able to hold back the currents, and HYCOM's
outages must not hold back ocean color. The two failure modes are unrelated
and should not share a publish gate.

One-way in the practical sense the sibling proved: moving a product between
repositories is cheap in machinery and expensive in everything that points at
it — roots in the contract, origins on the site, the union `check:docs` holds.
`PLAN.md`, "What this repository is for".

## D2 — 2026-08-30 — The default layer is a composite, not a single overpass

The owner's call, on a measurement rather than a preference.

Single-day coverage over a Chesapeake box, six consecutive days of S-3A:
19.9%, 12.6%, **0.0%**, 7.4%, 7.2%, 0.5% valid. **One day in six was
completely blank.** A single-scene layer would show a reader an empty map
about a third of the time, and would show them nothing on the day after a
storm — which is the day they most want to look.

One-way because it decides the shape of every published file: a composite
carries a window, and a window has to be in the file name, the header, and
the reader's understanding of what "now" means. Retrofitting a window onto a
single-scene contract is a migration.

**The composite LENGTH is deliberately not decided here.** Seven days is
recommended and measured; that recommendation is `PLAN.md`'s and the number
is open. It is a tuning decision inside this door, not another door.

## D3 — 2026-08-30 — The region is Sector FI whole: Florida to Nova Scotia

The owner asked to maximize coverage for the whole East Coast, and the
upstream happens to make that free: `noaacwS3AOLCIchlaSectorFIDaily` covers
lat 29.81–45.19, lon −80.04 to −59.96 in **one dataset at 278 m**. There is
no sector stitching to do, which is what would otherwise have set the
region's shape.

Recorded as a decision rather than a fact because the alternative was real —
a Chesapeake-only product, matching the owner's screenshot and the East Coast
Node's own `CB3` cut, would have been smaller and cheaper. It was rejected:
the same request serves the whole coast, and a bloom does not stop at a bay
mouth.

**What remains open is the PUBLISHED extent**, which is a storage question
against the Pages cap and is not this decision. Fetching the sector whole
does not commit to publishing it whole.

## D4 — 2026-08-30 — Merge Sentinel-3A and 3B rather than picking one

The owner's call, and the measurement supports it more strongly than expected.

Merging is worth roughly **a doubling of the single-satellite window**: seven
days of S-3A alone reaches 58.1% of what is ever observable, while five days
of the pair reaches 62.5%. At the seven-day window the merge is worth
**+16.1 points**, its largest gain.

One-way in the file shape, like D2: a merged product's provenance is
per-pixel, not per-file. A reader asking "which satellite saw this" needs an
answer the format has to carry, and adding that later is a migration.

## D5 — 2026-08-31 — The composite window is seven days

The owner's call, taking `PLAN.md`'s Recommendation 1 as measured.

Seven days is where the marginal return falls off, and it is the window
CoastWatch itself publishes for this region, so a reader comparing against
the official product compares like with like. Re-measured on the day of the
decision against a different window and a different denominator: **76.8% of
observable water**, where the founding measurement said 74.2%. Two
independent samples either side of three quarters is the strongest statement
this record can make about a number that moves with the season.

What the window costs, stated because it is the whole trade: the trailing
edge is a week old, and **a composite must not lie about its own age**. The
product therefore publishes a per-cell age beside the value — measured at
this window as median 2 days, p90 4, max 6.

One-way in the file shape: the window is baked into every published cell, and
changing it later re-dates every pixel a reader may have already acted on.
Widening it is cheap in machinery and expensive in trust.

## D6 — 2026-08-31 — Sector FI whole, at the instrument's own 278 m

The owner's call on a byte measurement, which is what `PLAN.md`'s Open item 2
was waiting for. **Published at native 0.0025° across the entire sector, with
no resolution gaps**, as a tile tier under a 0.05° whole-sector overview.

The numbers it was decided on, all measured 2026-08-31:

| | cells | published |
| --- | --- | --- |
| overview, 0.05° | 123,816 | 0.6 MB |
| tiles, 0.0025° (chosen) | 49,384,500 | ~234 MB |
| 0.005° considered | 12,346,125 | ~59 MB |
| 0.01° considered | 3,088,304 | ~15 MB |

**The argument against native turned out to be false, and it is recorded
because it nearly decided this the other way.** The concern was that a
198 MB single-frame request would time out and force a chunked fetcher. It
does not: the full 6150×8030 native frame returns in **10.7 s, 188.5 MB**,
about 17 MB/s, with no chunking. Measured before the design was written,
not after it failed.

**Values are stored linear at two decimals** — 4.98 bytes/cell. Chlorophyll
is log-distributed over 0.01–500 mg m⁻³, so the one-decimal encoding the
model fields use would flatten every open-ocean value to 0.1. Two decimals
keep 0.08 distinct from 0.09 and stop short of the third digit, which is
finer than the retrieval's own uncertainty. **Not log10**, though that is how
chlorophyll is conventionally drawn: a stored logarithm is a contract detail
every consumer must remember to undo, and the ones that forget fail quietly.

One-way on both counts, in the first sense this file names: resolution and
encoding are the shape readers code against. Coarsening later strands
anything built on the detail; changing the encoding silently changes every
number in every file.

## Open, and not decided here

Listed so nothing above is read as settling them: which HAB indicator to
build first (chlorophyll anomaly recommended), and whether to accumulate an
archive beyond the upstream's 90-day rolling window — and, since 2026-09-02,
whether the estuaries get a second origin (Copernicus Marine's OLCI product,
measured in `PLAN.md` section 5). All three are `PLAN.md`'s "Open" section.
**D5 and D6 closed the first two of the four that stood here**, on
2026-08-31.
