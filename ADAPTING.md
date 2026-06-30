# Adapting Permetrax to your jurisdiction

Permetrax is built so you can plug in **your own town's zoning rules without touching any
engine logic.** Everything is data: you add a jurisdiction object, fill in the numeric
limits and citations, and the rule engine, scoring, site plan, and explanations pick it up
automatically.

This guide walks through it end to end. Everything lives in `index.html`, in the
`JURISDICTIONS` object near the top of the `<script>` block.

---

## 1. The shape of a jurisdiction

```js
my_town: {
  id: "my_town",
  municipality: "City of Example",
  state: "CA",
  zone: "R-1 Single-Family Residential",
  dataset_version: "official · 2026",
  source_type: "official",          // "official" = real & cited; anything else = sample
  code_name: "Example Municipal Code § 12-3",
  code_url: "https://…/your-code",  // link shown in the results banner
  disclaimer: "Where these rules came from and what isn't modeled. Not legal advice.",
  rules: {
    // one entry per rule you want checked — see section 3
  }
}
```

Add your object inside `JURISDICTIONS` (the first entry becomes the default selection). The
dropdown is generated automatically; `source_type: "official"` gives it the green
“✓ Official code · cited” badge, anything else gives the muted “Sample data” badge.

## 2. The shape of a rule

```js
max_height_ft: {
  value: 30,                 // the numeric limit
  severity: "high",          // "high" | "medium" | "low"  (drives the score penalty)
  label: "Maximum building height",
  why: "Short plain-English reason the rule exists.",
  citation: {                // OPTIONAL — include for real, sourced rules
    code: "Example Muni. Code",
    section: "§ 12-3.4(b)",
    url: "https://…/code#anchor",
    quote: "No building shall exceed 30 feet",   // optional: exact text to highlight
    note: "Any caveat the simplified check doesn't model."  // optional
  }
}
```

- **Omit `citation`** and the UI shows “⚠ Sample dataset — illustrative rule, no official
  source” under that check. (That's how the fictional sample towns work.)
- **`quote`** enables the browser's text-fragment highlight (see section 5).
- **`severity`** controls scoring: `high` −25, `medium` −15, `low` −10 (see section 6).

## 3. The rule keys the engine understands

The engine only knows these specific keys. **Include the ones your code defines; omit the
rest** — an omitted rule is simply not checked.

| Rule key | What it compares | Needs lot size? |
|---|---|---|
| `max_height_ft` | building height ≤ value | no |
| `max_floors` | stories ≤ value | no |
| `setback_front_ft` | planned front setback ≥ value | no |
| `setback_side_ft` | planned side setback ≥ value | no |
| `setback_rear_ft` | planned rear setback ≥ value | no |
| `lot_coverage_max_percent` | footprint ÷ lot area ≤ value% | **yes** |
| `max_far` | (footprint × floors) ÷ lot area ≤ value | **yes** |
| `min_lot_area_sqft` | lot area ≥ value | **yes** |
| `min_lot_width_ft` | lot width ≥ value | **yes** |
| `max_impervious_percent` | (footprint + driveway/patio) ÷ lot area ≤ value% | **yes** |
| `min_parking_spaces` | off-street spaces ≥ value | no |

Rules marked “needs lot size” only run when the user enters lot width and depth; otherwise
they're reported as a warning. Setback and parking rules become warnings when the user
leaves those inputs blank.

> Want a brand-new kind of rule (e.g. max building length)? That's the one case that needs a
> code change — add a check in the `analyze()` function following the existing patterns
> (`vMax` / `vMin` / `vSetback`), then add a matching `fixFor()` entry.

## 4. Step by step: add a real jurisdiction

1. **Find your code online.** Many US codes live on eCode360, Municode, or American Legal.
   Locate the single-family / residential district's dimensional standards.
2. **Read the actual numbers** — height, stories, setbacks, coverage, FAR, lot minimums,
   parking, impervious. Only include what the code actually specifies.
3. **Copy a jurisdiction block** (section 1) and fill in the metadata.
4. **Add one rule per standard** (section 2), using the keys in section 3.
5. **Grab a deep link per rule.** On eCode360, each subsection has its own URL — click the
   subsection's paragraph symbol / “link” icon to get a URL like
   `https://ecode360.com/12345678#12345678`. Put that in `citation.url`.
6. **(Optional) Add a `quote`** — copy the exact sentence from the code so the link
   highlights it (section 5).
7. **Save `index.html` and open it.** Your jurisdiction is in the dropdown and fully wired.

## 5. Clickable, self-highlighting citations

`citation.url` is the link. If you also add `citation.quote` with text copied verbatim from
the page, Permetrax appends a **text fragment** (`#:~:text=…`) so a modern browser scrolls to
**and highlights** that exact sentence.

- The `quote` must match the page text closely (matching is case-insensitive).
- It degrades gracefully: browsers without text-fragment support still land on the section
  via the regular anchor, and sites that don't expose the text just open at the top.
- For PDF-only codes, you can instead deep-link to a page with `…/code.pdf#page=47`.

## 6. Tuning scoring & ratings (optional)

Near the top of the script:

```js
const SEVERITY_PENALTY = { high: 25, medium: 15, low: 10 };  // points off per violation
const WARNING_PENALTY  = 4;                                  // points off per warning
```

The 0–100 score starts at 100 and subtracts these. The permit-likelihood wording lives in
the `likelihood()` function (thresholds 85 / 65 / 40) — adjust to taste.

## 7. Ship only your jurisdiction

To remove the fictional sample towns, delete the `maple_grove`, `lakeside`, and `brookfield`
objects from `JURISDICTIONS`. Keep at least one jurisdiction. Update `SAMPLE` (the
“Load sample” preset) to use your `id` and realistic numbers.

## 8. Deploy your fork

It's a single static file — host it anywhere:

- **Netlify Drop:** drag the folder (with `index.html`) onto https://app.netlify.com/drop.
- **GitHub Pages:** push to a repo → Settings → Pages → deploy from `main` / root.

---

Questions or improvements welcome — it's MIT licensed, so fork freely.
