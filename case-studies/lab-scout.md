# Lab Scout

**A deal radar for the used lab-equipment market.** Watches eBay, dealer sites
and auction feeds, extracts structured records with an LLM, scores them for
resale margin against hard buy-box gates, and pushes a ranked digest and instant
Telegram alerts to a human who makes every actual decision.

`Python · Claude · SQLite · Playwright · Telegram`

---

## The problem

The UK/EU used lab-equipment market is genuinely fragmented. Stock sits across
eBay, a handful of specialist dealers, Shopify storefronts, JS-heavy
marketplaces and auction houses, each with its own format and none with a usable
API. Listings are free text written by people who don't know what they have — a
£30k liquid handler described in two lines with no mention of whether the
software licence, the pipetting head or the deck is included.

The margin in that market is entirely in knowing what a listing actually *is*
and what it's actually worth, faster than the next person. That's a
domain-knowledge problem wearing a data-engineering costume.

## Scout, not trader

The single most important design decision: **there are no buy actions anywhere
in the codebase.** No bidding, no offers, no transactions. It surfaces ranked
candidates with a verdict and a reason; a human commits the money.

This isn't timidity. It's that the failure mode of an autonomous buying agent in
an illiquid market is buying something unsellable, and the cost of that is
unbounded while the benefit of removing the human is a few minutes. The
human-in-the-loop boundary is the product, not a limitation of it.

## Pipeline

```
ingest → extract → score → match → digest
```

**Ingest.** Four source types, each with its own adapter: the eBay Browse API,
HTML dealer sites, Shopify JSON feeds, and JS/anti-bot marketplaces via
Playwright. Auction houses are a register-and-subscribe checklist rather than a
scrape, because scraping them is both harder and less legal. Sites that stay
blocked even with a real browser are documented as disabled rather than
silently failing. Every fetched payload is snapshotted to `data/raw/`, so the
history is re-extractable when the extraction prompt improves.

**Extract.** A versioned Claude prompt turns listing free text into a structured
record — brand, model, category, completeness, condition, what's missing — then
an enrichment pass maps it to a canonical model and sets support flags.

**Score.** Estimated resale from comparable sales, minus buy cost, freight and
import costs, adjusted by a generation penalty and a substitutability factor.

**Gate.** Separately from the score, a buy-box evaluates hard rejects.

**Match.** Two further modes: demand-led sourcing against committed customer
projects (only projects that actually exist — sell before you buy), and a
component hunt against a target build.

**Alert.** A ranked digest plus instant Telegram pings. Every integration fails
soft: no Telegram token means it prints to the console rather than crashing the
nightly run.

## The gates are where the domain knowledge lives

This is the part I'd point at. From the top of `buybox.py`:

> *Every gate encodes an expensive lesson: Lynx (broken isn't cheap), Genesis
> (obsolete isn't cheap), completeness (unverifiable is worse than faulty).*

Each named gate is a loss someone actually took, turned into a rule:

- **Spares-or-repairs in a safety-critical category** — an automatic reject. A
  cheap broken unit in a category where refurbishment is regulated isn't cheap.
- **The obsolete-platform gate** — software-controlled instrument, two-plus
  generations behind, dead vendor ecosystem. The hardware works; nothing can
  drive it.
- **Completeness** — a liquid handler whose hardware can't be verified as
  complete is rejected, *unless* the missing pieces are ones an open-source
  control stack can substitute. That carve-out is the actual commercial edge,
  and it's a config entry rather than an `if` statement.
- **Centrifuge without a rotor**, **collection-only outside the serviceable
  region**, **price outside the model's band**.

A real run: **941 listings scored → 12 pass, 592 flagged for review, 337
rejected.** A 1.3% pass rate is the system working. The value is in the 337
rejections, each of which carries its reason, so the operator can audit the
judgement rather than trust it.

## Two things I'd defend in an interview

**The strategy lives in config, not in code.** All thresholds, gates,
equivalence classes, target models and source definitions are YAML — about 690
lines of it against 3,400 lines of Python. When market conditions change you
edit a file, not a function. It also means the operator can read the strategy
without reading Python, which matters when the operator is the domain expert and
the domain expert isn't a programmer.

**Regional comps are isolated.** US listings are fully modelled as
buy-and-import candidates, with trans-Atlantic freight by size class, duty,
handling and optional VAT added to the buy cost before margin. But **US prices
are excluded from the UK resale estimate** by default. A US instrument listed at
£30k must not be allowed to set the UK resale value of the same instrument —
that one line of config is the difference between a working margin model and a
confidently wrong one. Realised sales always count, wherever they happened.

## The query bot

The pipeline pushes; the bot pulls. Plain-English Telegram messages —
*"hamilton starlet under 15k"*, *"UK plate reader complete"* — are parsed by
Claude into a structured filter (brand, model, category, price, region,
completeness) and run against current listings. If the API is unavailable a
keyword parser takes over, degraded but alive.

Results show the buy-box verdict inline — ✅ buy / ⚑ review / ✗ rejected with
the reason — so a query returns everything that matches, not just the clean
buys. Seeing *why* something was rejected is how the operator builds trust in
the gates, and how they spot a gate that's wrong.

The bot is locked to a single chat ID.

## Shape of it

- **~3,400 lines of Python** across ingest / extract / score / match / alert /
  query, plus a CLI.
- **~720 lines of tests** across 12 files, including three fixtures the build
  plan mandated for specific failure cases.
- **~690 lines of YAML** carrying the entire strategy.
- **572 listings and 3,239 scores** in the SQLite database, which is designed to
  compound — raw snapshots retained so old listings can be re-extracted against
  a better prompt.
- Nightly scheduled run; secrets fail soft when absent.

---

*Personal project. Source is private — it contains live source definitions and
the buy-box thresholds, which are the commercially useful part.*
