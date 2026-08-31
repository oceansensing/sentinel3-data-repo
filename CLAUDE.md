# sentinel3-data-repo

Sentinel-3 OLCI ocean color for the US East Coast, and the harmful algal
bloom indicators built on it. A sibling data repository: it publishes to
GitHub Pages on its own cron, holds no code of its own, and runs the shared
orchestrator against its own `pipeline/products.toml`.

**Nothing is built yet.** This repository currently holds its founding plan
and nothing that runs. `PLAN.md` carries the feasibility work — the upstream,
the measurements that decide the product's shape, and what is still open.
Read its "What was measured" before trusting any number anywhere.

<!-- DOC-DOCTRINE v1 begin — identical in all five repositories; `check:docs` holds them equal. Edit one, sync all. -->
## Where truth lives, and what "update docs" means

Five repositories carry this project: `oceanlet.js` (the engine),
`oceansensing.github.io` (the site, and every fetch script),
`realtime-data-repo` (the orchestrator every data repository runs, and most
products), `espc-model-repo` (the ESPC currents), and `sentinel3-data-repo`
(Sentinel-3 ocean color and HAB indicators, added 2026-08-30). Each document
answers exactly one question.

`eccofs-model-repo` was created on 2026-08-30 for a future ECCOFS layer and
is **empty**. It is not in the list above and is not swept, because there is
nothing yet to be false. **It joins the list in the commit that gives it
documents** — naming it here so that moment is not missed, which is the
lesson the sixth repository below cost.

**They are MEANT to carry the same four documents and three of them do not**
— only `oceanlet.js` and `sentinel3-data-repo` have a `DECISIONS.md`
(measured 2026-08-30). That is a gap in the repositories, not a license to
skip the file: a data repository closes one-way doors too, and it has
nowhere to say so.

| file | answers | tense | it is stale when |
| --- | --- | --- | --- |
| `README.md` | what this is, how to run it | present | a reader types a command or trusts a number and is wrong |
| `CLAUDE.md` | what must not be got wrong here | imperative | the next session is about to repeat a mistake |
| `PLAN.md` | what happened, measured, and what is open | dated past | "why is it like this?" has no answer here |
| `DECISIONS.md` | which one-way door closed, and when | dated | a reversal would cost a migration and nothing says so |
| `docs/` | contracts, ledgers and the guide | present | it describes an interface, a divergence or a concept that has moved on |

**`docs/` is a first-class part of "all docs", not an appendix** — the owner
asked for that explicitly on 2026-08-28, and the reason is that these are the
documents everything else points AT. A frozen contract, a divergence ledger
whose rows are pinned by tests, a guide that introduces the model: each is
the thing a reader is sent to when the short answer will not do, so each is
the worst place for a claim that has quietly stopped being true.

**"Update docs" means a sweep of all five repositories, not the one in hand.**
Docs are part of the change, never a follow-up and never a separate ask. Six
questions, asked of every repository the change touched:

1. Did a command, a path, a script name or a number a reader would type or
   trust move? → `README.md`
2. Did a rule, a trap, or a things-that-must-move-together change or come to
   light? → `CLAUDE.md`
3. Did something *happen* — a measurement, a defect, a yield, a mechanism, an
   open question opened or answered? → `PLAN.md`
4. Did a one-way door close? → `DECISIONS.md`
5. Did an interface, a deliberate divergence, or a concept the guide explains
   move? → the matching file under `docs/`
6. **Does a document in another repository now say something false because of
   this change?** → fix it there, in the same sitting.

**Question 6 is the one that gets missed, and it is why this block is
identical in five places.** Measured 2026-08-28: one tile-tier measurement
falsified `espc-model-repo`'s README, its `products.toml` header and the
site's README at once. Two were found; the third took a reminder from the
owner, who then asked for this doctrine.

**A SIXTH repository consumes this system and is deliberately NOT in the five
above**: `ocean-now`, the iOS port, which mirrors the site's published
contract. It is not swept by these six questions and does not carry this
block — it has a lighter mechanism instead, a pending list in its parity
ledger, and the two repositories whose changes can reach it (the engine and
the site) each say so in their own section. It is named here because "four"
was read as "all of them" for two weeks while that ledger drifted 176 commits
behind with nothing noticing — which is question 6 failing at the granularity
of a whole repository rather than a document. Adding a repository to the list
above is therefore a real act: it buys the sweep, and leaving one off costs
exactly what that cost.

A number in prose is only as good as its anchor. `check:docs` gates every
claim it can tie to a source constant and nothing else, so when a figure has
no anchor — a measurement, a live reading, a byte count off a build log —
write **where it was measured and when**, or the next reader cannot tell a
fact from a guess that aged.
<!-- DOC-DOCTRINE v1 end -->
### In this repository

- **`PLAN.md`** — the founding plan and the running record: the upstream and
  its contract traps, the coverage experiment behind the composite, and the
  open questions. It is where a measurement goes.
- **`README.md`** — what this is and how to run it, once there is something
  to run.
- **`DECISIONS.md`** — the dated one-way decisions, D1 onward.
- **`pipeline/products.toml`** — what this repository publishes. Not written
  yet.

## What must not be got wrong here

### The upstream has three traps, and each was found by a request failing

Not by reading documentation. A fetcher written from the catalog alone will
hit all three.

1. **The grid is 4-D: `time, altitude, latitude, longitude`.** Omitting
   `altitude` does not fail cleanly — ERDDAP maps the constraints onto the
   wrong axes and returns a 404 saying a latitude exceeds the altitude
   maximum. Pass `[(0.0):1:(0.0)]`.
2. **Latitude DESCENDS** (45.186 first). Ranges are given north-first.
3. **ERDDAP 403s the default Python `urllib` user agent.** The failure is a
   bare `HTTP Error 403: Forbidden` with no hint. `curl` is fine; `requests`
   with an explicit UA is fine.

### A single scene is not a product, and the composite must say its age

Measured 2026-08-30 over a Chesapeake box, six consecutive days of S-3A:
19.9%, 12.6%, **0.0%**, 7.4%, 7.2%, 0.5% valid. **One day in six was
completely blank.** Never ship a single-overpass layer as the default; the
reader who most needs it will look on the blank day.

And a composite hides which day each pixel came from. **Publish a per-cell
age beside the value** — the currency gate's argument at pixel granularity.
A bloom whose freshest pixel is six days old must be able to say so.

**Median, not mean, and not latest.** The median survives an undetected thin
cloud or a sun-glint edge; a mean is dragged by exactly the outliers that are
most often not water.

### A title is not a measurement

CoastWatch's `noaacwecnOLCImultisensorCHLeastcoast7Day` is titled
"2016-present" and its `time_coverage_end` is **2025-05-27** — over a year
behind, measured 2026-08-30. It is not a source here, but it is the reason
**the currency gate points at this upstream from the first published file**.
Do not infer freshness from a dataset's name, its title, or its summary; read
`time_coverage_end`, and publish what you read.

### The upstream keeps 90 days, and that is the whole archive

`noaacwS3AOLCIchlaSectorFIDaily` and its S-3B twin are **90-day rolling
windows**. Anything this repository wants to say about a season — and any
anomaly baseline built from its own history rather than a published
climatology — has to be accumulated HERE. There is no going back for it
later.

### Two rules inherited from the sibling data repositories

Both learned the hard way in `realtime-data-repo` and `espc-model-repo`, and
neither is optional here:

- **Every `roots` entry in `products.toml` must be one the site's
  `test-schema.mjs --roots` publishes.** The orchestrator exits 2 on a root
  the contract does not know and stops the publish. It does not fail on the
  reverse; the union across origins is held by the site's `check:docs`, the
  only side that can see every declaration at once.
- **A product that leaves takes its files with it.** The stage is seeded from
  what is already published, so a withdrawn product lingers unless it is
  removed deliberately.

### Do not scrape the East Coast Node

Their daily Chesapeake `chl_switch` product — 300 m, a switching algorithm
built for turbid water that standard `chlor_a` is not — is **not on ERDDAP**,
and `eastcoast.coastwatch.noaa.gov/data_access.php` 404s. If that algorithm
turns out to beat `chlor_a` inside the Bay, **that is an email to the node,
not a scraper.** A hand-built scraper against a page that already 404s is a
dependency on someone's directory layout.

### The orchestrator is shared, so a change to it is a change to every repository

`pipeline/orchestrate.py` lives in `realtime-data-repo` and every data
repository runs it, pointed at its own workspace through `PIPELINE_ROOT`. The
fetchers and the data contract live in `oceansensing.github.io` and are
checked out at run time, so a fetcher or `schema.ts` change lands here on the
**next run**, not on any push here.

## The working agreement

The same one the sibling repositories keep, in short: a measured constant
moves with its reason in the same commit; new checks are mutation-tested
before they are believed; exit codes are captured before output is read; docs
are part of the change, never a follow-up. **A number in prose without an
anchor says where it was measured and when, or it is a guess that will age.**
