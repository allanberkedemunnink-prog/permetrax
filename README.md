# Permetrax

**Educational** pre-submission zoning & building-code checker for residential projects.

Describe a building concept (height, footprint, floors, lot, setbacks) and Permetrax
compares it against a jurisdiction's zoning rules — flagging likely violations, drawing a
to-scale site plan, scoring permit likelihood, and explaining everything in plain English.

> ⚠️ **Not legal advice.** Permetrax runs against *simplified, fictional sample datasets*,
> not any real municipal code. Always verify against the official code and a licensed
> professional before submitting permits.

**Live site:** https://allanberkedemunnink-prog.github.io/permetrax/

## How it works

The key design decision: **the rule engine is deterministic — no AI decides compliance.**
All pass/fail logic and math run in plain JavaScript so results are reproducible and
auditable. An optional LLM layer (Claude) only *explains* the engine's output — it never
invents rules or changes a verdict. With no API key, a built-in explanation engine is used,
so the app is fully functional offline.

## Citations

Every check links to its source. For the real jurisdiction, each rule shows the exact
code section (e.g. *§ 10-2.2503(b)*) as a **clickable citation that opens the actual
ordinance at that section** on eCode360. Sample jurisdictions are clearly labeled as
illustrative (no official source).

## What it checks

Jurisdictions:

- **★ City of Redondo Beach, CA** — R-1 Single-Family — **real code, every rule cited**
  (transcribed from the Redondo Beach Municipal Code § 10-2.2503 on eCode360)
- **Township of Maple Grove, NJ** — R-1 Single-Family *(fictional sample)*
- **City of Lakeside, CA** — R-2 Low-Density *(fictional sample)*
- **Town of Brookfield, TX** — SF-2 Suburban *(fictional sample)*

Rules: max height, max stories, front/side/rear setbacks, lot coverage, floor-area ratio
(FAR), minimum lot area & width, off-street parking, and impervious coverage (which rules
apply depends on the jurisdiction).

## Run it

It's a single self-contained file — no build, no server.

- **Online:** open the live site above, or
- **Locally:** download `index.html` and double-click it.

Click **Load sample** → **Run compliance check**.

## Tech

Vanilla HTML/CSS/JS, zero dependencies. Deterministic rule engine + SVG site plan +
`localStorage` saved projects + print-to-PDF export.
