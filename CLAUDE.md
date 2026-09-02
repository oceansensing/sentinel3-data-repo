# sentinel3-data-repo

Sentinel-3 OLCI ocean color for the US East Coast, and the harmful algal
bloom indicators built on it. A sibling data repository: it publishes to
GitHub Pages on its own cron, holds no code of its own, and runs the shared
orchestrator against its own `pipeline/products.toml`.

**It publishes, since 2026-08-31**, from dispatched runs while the crons stay
commented out. `PLAN.md` carries the feasibility work — the upstream,
the measurements that decide the product's shape, and what is still open.
Read its "What was measured" before trusting any number anywhere.

<!-- DOC-DOCTRINE v1 begin — identical in all ten repositories; `check:docs` holds them equal. Edit one, sync all. -->
## Where truth lives, and what "update docs" means

Ten repositories carry this project. The engine and the site:
`oceanlet.js`, `oceansensing.github.io` (the site, and every fetch script).
The orchestrator and the observations: `realtime-data-repo`. And the data
repositories, which since 2026-08-30 split **currents from fields** per model:
`espc-model-repo` (the ESPC currents — a legacy name, see below),
`espc-model-fields-repo`, `eccofs-model-currents-repo`,
`eccofs-model-fields-repo`, `mercator-model-currents-repo`,
`mercator-model-fields-repo`, and `sentinel3-data-repo` (ocean color, which
has no vector half to split). Each document answers exactly one question.

**`espc-model-repo` is the ESPC CURRENTS repository** despite its name — the
one exception to the convention, kept because its URL is a live origin and
GitHub Pages does not reliably redirect a renamed project site. Read it and
`eccofs-model-currents-repo` as the same kind of thing.

*(`eccofs-model-repo` was RENAMED to `eccofs-model-fields-repo` on 2026-08-30,
not superseded — GitHub redirects the old name, which is why a rename was
free there and is not free for `espc-model-repo`: that one has published
bytes behind a Pages URL, and Pages does not redirect what the API does.)*

**All ten carry the same four documents, and since 2026-08-31 a gate holds
them to it** — `check:docs` requires a `DECISIONS.md` tracked in git in every
repository. The last two landed that day, the site's and
`realtime-data-repo`'s, reconstructed from records that already existed:
nothing was missing but the file, which is how the site went seven weeks
without one and `realtime-data-repo` eighteen days. **This block asserted
otherwise from the day it was written** — byte-compared in the eight places there were then, and
false in two of them, because a gate on a text is a gate on the text. What it
cost is measurable: the engine promotion's own rehearsal listed *"a dated
entry in this repo's decisions and oceanlet's"* as its ninth step, and the
half with nowhere to go was simply not written.

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

**"Update docs" means a sweep of all ten repositories, not the one in hand.**
Docs are part of the change, never a follow-up and never a separate ask. Six
questions, asked of every repository the change touched:

1. Did a command, a path, a script name or a number a reader would type or
   trust move? → `README.md`
2. Did a rule, a trap, or a things-that-must-move-together change or come to
   light? → `CLAUDE.md`
3. Did something *happen* — a measurement, a defect, a yield, a mechanism, an
   open question opened or answered? → `PLAN.md`
4. Did a one-way door close — **or has one already recorded stopped being
   fully true**? → `DECISIONS.md`, in **every** repository the change
   touched. All ten carry one, so this is no longer the
   engine's question with seven exemptions; the amendment half is here
   because two entries needed one within a day of being written.
5. Did an interface, a deliberate divergence, or a concept the guide explains
   move? → the matching file under `docs/`
6. **Does a document in another repository now say something false because of
   this change?** → fix it there, in the same sitting.

**Question 6 is the one that gets missed, and it is why this block is
identical in ten places.** Measured 2026-08-28: one tile-tier measurement
falsified `espc-model-repo`'s README, its `products.toml` header and the
site's README at once. Two were found; the third took a reminder from the
owner, who then asked for this doctrine.

**Two repositories are deliberately NOT in the list above, on opposite
grounds, and both are named because an exclusion nobody wrote down is
indistinguishable from an oversight.**

`ocean-now`, the iOS port, **consumes this system** — it mirrors the site's
published contract. It is not swept by these six questions and does not carry
this block; it has a lighter mechanism instead, a pending list in its parity
ledger, and the two repositories whose changes can reach it (the engine and
the site) each say so in their own section. It is named here because "four"
was read as "all of them" for two weeks while that ledger drifted 176 commits
behind with nothing noticing — question 6 failing at the granularity of a
whole repository rather than a document.

`hab-data-repo` is excluded on the opposite ground: **it does not touch the
ocean map at all** (the owner's call, 2026-08-31). It publishes the bloom
photographs for a different part of the website, reached through `HAB_DATA`
in `src/config.ts`, and carries no interface anything here codes against
beyond a URL and a filename convention. It needs no mechanism, not even a
lighter one — nothing in these ten can falsify a claim in it, and it cannot
falsify one here. Do not mix it in.

Adding a repository to the list above is therefore a real act: it buys the
sweep, and leaving one off **silently** costs exactly what `ocean-now` cost.

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
- **`README.md`** — what this is and how to run it.
- **`DECISIONS.md`** — the dated one-way decisions, D1 onward.
- **`pipeline/products.toml`** — what this repository publishes: one
  product, two roots, two tile tiers.

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

### Three rules inherited from the sibling data repositories

Both learned the hard way in `realtime-data-repo` and `espc-model-repo`, and
neither is optional here:

- **Every `roots` entry in `products.toml` must be one the site's
  `test-schema.mjs --roots` publishes.** The orchestrator exits 2 on a root
  the contract does not know and stops the publish. It does not fail on the
  reverse; the union across origins is held by the site's `check:docs`, the
  only side that can see every declaration at once.
- **A product that leaves takes its files with it.** The stage is seeded from
  what is already published, so a withdrawn product can linger and be served
  frozen. Measured 2026-08-31: undeclaring a whole product self-heals in two
  runs, because the `published` branch is assembled from the declared products
  and the Pages tree from the stage. **Renaming a file inside a product that
  still exists does not heal at all** — its `writes` glob still matches, so
  the bank keeps it. That is the case that has actually cost bytes.
- **A step's scope must match its products', and it is one change, never
  two.** Added 2026-08-31, learned by a failed production run. If a fetch
  script publishes several families and this repository owns only some of
  them, the `[steps.*]` `cmd` needs `--only=` naming exactly what its products
  declare — the per-product `namespace`, `tiles` and `tile-key` commands
  being scoped is not enough. Too WIDE and the step writes files no product
  declares and the write fence refuses the whole run; too NARROW and files
  somebody declared are never written and the previous copies carry forward
  frozen, silently. A *product* is the unit of ownership; a *step* is the unit
  of execution. The site's `check:docs` holds both directions across every
  origin.

### Do not scrape the East Coast Node

Their daily Chesapeake `chl_switch` product — 300 m, a switching algorithm
built for turbid water that standard `chlor_a` is not — is **not on ERDDAP**,
and `eastcoast.coastwatch.noaa.gov/data_access.php` 404s. If that algorithm
turns out to beat `chlor_a` inside the Bay, **that is an email to the node,
not a scraper.** A hand-built scraper against a page that already 404s is a
dependency on someone's directory layout.

### The estuaries are empty upstream, not here

Measured 2026-09-02, after the owner asked whether a quality flag was hiding
the Chesapeake's tributaries: **the fetcher masks nothing but the 0–500 range,
the published tiles and overview match a rebuild from the upstream 100% cell
for cell, and the upstream serves one variable and no flags.** The James, York
and Potomac carry 1–5% of their pixels on a day the Bay mouth carries 76%,
because OC4Me's retrieval fails within about 5 km of every shore. Do not go
looking for a flag in this pipeline, and do not "fix" it by widening the
composite — a fortnight of nothing is still nothing. `PLAN.md` section 4 has
the measurement and section 5 the alternatives; the one that measured better
is Copernicus Marine's OLCI product, whose shore-loss zone is about one pixel.

**If a Copernicus Marine fetcher is ever written for this repository, it
inherits the Mercator fetcher's masking rule.** This repository is PUBLIC and
the credentials are a username and password held as secrets; the toolbox is
the only thing that reads them, and every line it prints goes through the
mask that `fetch-mercator.py` gates with `mask_self_test`. A log line with a
username in it is a leak on a public origin.

### The orchestrator is shared, so a change to it is a change to every repository

`pipeline/orchestrate.py` lives in `realtime-data-repo` and every data
repository runs it, pointed at its own workspace through `PIPELINE_ROOT`. The
fetchers and the data contract live in `oceansensing.github.io` and are
checked out at run time, so a fetcher or `schema.ts` change lands here on the
**next run**, not on any push here.

### A coarser cadence is three changes, never one

**Learned 2026-08-31 in `espc-model-fields-repo`**, whose heat-content layer
went from the shared 3-hourly hour to 6-hourly to halve a tile tier's
bandwidth. It is the closest thing this repository will do to a composite's
cadence, so the shape is worth having before it is needed here.

Publishing less often than your siblings touches three things, and only one of
them is the cadence:

1. **The cadence itself** — and express it against the CLOCK, not against a
   run count. "Every other run" drifts, because GitHub delivers scheduled runs
   45 min to 4 h 19 apart; "only when the hour is a multiple of N" cannot.
2. **`max_age_hours`**, or the currency gate marks the product `behind` on
   every run and fails the workflow after every deploy. A cadence and its
   staleness budget are one decision.
3. **Any cross-product rule in the site's contract that assumes everything
   moves together.** ESPC's hour rule treats a same-run hour mismatch as a
   **quarantine** — it withdraws the layer and ships the previous copy, with
   every gate green. The fix was to teach the rule from a published header
   field rather than exempt the product, so the guarantee got weaker in a
   stated, checkable way instead of silently.

**A composite has this problem by construction**: a 7-day window is a cadence
of days against neighbours on hours, and whatever this repository publishes
beside it will need the same three answers.

### Finder's `.DS_Store` is ignored here, and was tracked until 2026-08-31

This repository had **no `.gitignore` at all** until then, so macOS's
`.DS_Store` was an ordinary versioned file — six of them across the five data
repositories, all removed from the index and ignored that day. The copy on
disk is left alone; Finder owns it and rewrites it on the next visit.

**Every one of the six arrived on a documentation commit**, and none was
deliberate. This repository's rode `32d63e7` (2026-08-31), one of three that
took theirs under the same message — *A coarser cadence is three changes,
never one*. A cross-repository doc sweep is `git add -A` run in five
repositories in one afternoon, so a file none of them ignored entered four of
them on a single day — **the doctrine's own sweep was the vector.** The three
code repositories were never exposed to it, having ignored `.DS_Store` since
their first commit or the day after.

What it cost is that **`git status --porcelain` stopped being an answer.**
Finder rewrites a tracked `.DS_Store` whenever the directory is opened, so the
tree read dirty from a window rather than from an edit — and "is this tree
clean before I push" is only worth asking when a dirty tree means something.

It is also the file class behind the engine repository's 2026-08-30 fault, one
step earlier. There a `git rm -r` left a `.DS_Store` behind, so the emptied
directory still existed on disk and `existsSync` path claims went on resolving
locally while a fresh clone had nothing — green locally, red on CI, which is
why `check:docs` asks git rather than the filesystem. **A tracked `.DS_Store`
is the same disagreement between a clone and a working tree, in a file nobody
chose to version.**

**An ignore rule never untracks what is already in the index**, which is why
the fix here was `git rm --cached` and not a `.gitignore` line alone. A global
`core.excludesFile` covering the whole Finder family was written on this
machine at 13:13 on 2026-08-31, on the owner's instruction — *always by
default gitignore them and never track them* — and the last of the four
additions of that day landed at 13:02: eleven minutes too late to prevent
them, and structurally unable to reverse them.

**That global file is machine-local, so the `.gitignore` here is the half a
clone gets**, and it is not redundant with it. Blank it under `git -c
core.excludesFile=/dev/null` and the tree goes dirty with exactly the files
this section is about; restore it and the tree is clean. That is how the rule
was checked rather than assumed — with the global rule left on, blanking this
file changes nothing and the test sees nothing.

## The working agreement

The same one the sibling repositories keep, in short: a measured constant
moves with its reason in the same commit; new checks are mutation-tested
before they are believed; exit codes are captured before output is read; docs
are part of the change, never a follow-up. **A number in prose without an
anchor says where it was measured and when, or it is a guess that will age.**
