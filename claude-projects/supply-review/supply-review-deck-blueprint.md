# Supply Review — Deck Blueprint (Text Design)

**Cycle:** July 2026 — first cycle under Supply Planning ownership
**Owner / presenter:** Ben Elphick
**Forum:** Monthly S&OP, scheduled after the Demand Review (DR)
**Theme:** Openway (match existing Status Update deck — less-is-more, facts / KPIs, one message per slide)

---

## Governing methodology (Oliver Wight — *The Supply Review*, paper 4 of 7)

- The Supply Review is the **third step in the monthly IBP process**, after the Demand Review — supply's **forward-looking, time-phased** response, reporting performance to budget **past and (most importantly) projected**, and **documenting the assumptions, opportunities and vulnerabilities** around the forward plan (Intro, p2; p5).
- Structured on the **four parameters** (p11): **Monitor · Align · Measure · Adapt**.
- Serves the three supply priorities (p4): **product availability · right cost · working capital (cash)**.
- **Horizon:** minimum 24 months, ideally 36 → to **FY28** (fiscal year starts July).
- **Maturity note:** low maturity, no ITP yet, so this deck carries both the monthly look-back and the forward plan. Where data or policy isn't yet in place, the slide **shows the gap as a documented item on the journey** — facts, not excuse.

## Audience / decision rights

- **Required (decision audience):** Jim De Fegely · Sofia Christopoulos · Fiona Bull · Bernadette McDougall · Sam Bevacqua · David Richardson · Fergus Wilson
- **Optional:** Andre Roberts · Bernard Mutua · Fulbert Xavier · Hiral Dattani · Braulio Aedo · Matty Buhagiar · Lee Mutch
- **Owner:** Ben Elphick

## Deck arc

Title/frame → **Monitor** (what happened / where we are) → **Align** (against policy, service, the demand plan) → **Measure** (KPI scorecard) → **Adapt** (forward capacity, the route-integrity vulnerability, actions) → decisions & roadmap.

---

# Slide-by-slide

### 1 — Title
- Supply Review · July 2026 Cycle · Supply Planning · Ben Elphick.
- Sub-line: "Third step of the IBP cycle — supply's forward response to the agreed demand plan."
- *Priority: —  | Parameter: —*

### 2 — Purpose & how to read this deck
- **Message:** what the SR is, and the frame the deck follows.
- One line on IBP position (after DR, feeds Integrated Reconciliation).
- The four parameters shown as the deck's spine: Monitor / Align / Measure / Adapt.
- Horizon to FY28; priorities availability / cost / cash.
- One line: "First cycle under Supply Planning ownership — maturing; gaps are flagged, not hidden."
- *Priority: all | Parameter: frame*

### 3 — Story of supply: what changed since the Demand Review
- **Message:** the narrative hook OW calls for — how the demand plan moved and how supply is responding (build stock here, reserve capacity there).
- 3–4 bullets max: key demand shifts this cycle, supply's response, any imbalance carried forward.
- *Priority: availability | Parameter: Monitor | Source: DR output + supply plan*

### 4 — Production schedule adherence (management item 1)
- **Message:** execution health for the month — did we make what we planned, when.
- Headline adherence % (orders completed to schedule); trend vs prior month.
- Adherence by line / production pool (bar).
- Major misses called out (top 3–5), each with a one-line cause.
- "Opportunities for improvement" — 2–3 concrete, no narrative.
- *Priority: availability | Parameter: Monitor + Measure | Source: D365 batch order header — Scheduled vs Started/Ended/RAF dates. NB this is **date** adherence; **hours/route** adherence is Slide 9.*

### 5 — Total inventory position (management item 3)
- **Message:** how much cash is tied up, and where.
- Total inventory $ split Raws / Packaging / FG; trend line.
- Days of cover / DIO by segment.
- Callout: any segment moving materially against plan.
- *Priority: cash | Parameter: Monitor | Source: D365 on-hand valuation (source view TBC)*

### 6 — Stock on hand vs policy — by family (management item 2)
- **Message:** are the priority families where policy says they should be.
- Oats · Rice Cakes · Granolas — SOH vs policy target (position vs band).
- RAG per family; days of cover.
- **Maturity flag:** if policy targets aren't yet defined for a family, show **position only** and flag "policy target to be set" as an action → carries to Slide 12. Do not invent a target.
- *Priority: availability + cash | Parameter: Align | Source: D365 on-hand by family; policy targets (status per family TBC)*

### 7 — KPI scorecard (the Measure layer, surfaced)
- **Message:** one-page read of supply health against KPIs.
- Tiles: schedule adherence % · SOH vs policy (3 families) · total inventory $ & trend · planning-rate status · **route-integrity flag**. RAG + arrow.
- Each tile = number, target, direction. No prose.
- *Priority: all | Parameter: Measure | Source: consolidates slides 4–6, 9, 10*

### 8 — Capacity outlook — FY27 / FY28, all lines (management item 4)
- **Message:** where the forward load meets or breaches capacity across the horizon.
- Load vs available capacity by line, time-phased to FY28; highlight constraint lines.
- Named risks / assumptions on the forward view.
- **Dependency flag (critical):** this view is generated from the **RCCP master plan, which is currently parked (register priority #1).** Until it's stood up, this slide is **provisional / partial** — state that explicitly on the slide.
- *Priority: availability + cost | Parameter: Adapt (Monitor of forward capacity) | Source: RCCP plan (pending) + Capacity Reserved measure (existing pbix)*

### 9 — Route integrity: planned vs actual line time  *(paramount thread)*
- **Message:** the forward capacity plan rests on route standards that don't match reality — here is the gap, its impact on the capacity outlook, and the path to close it. **Facts, not excuse.**
- Live worked example (BO013235, Packout): route standard **65.4h** → D365 machine "actual" **171h (contaminated — one open span across a long weekend)** → **labour-union active time ~60h**.
- The method in one line: active line time = **union** of labour clock intervals (≥1 clocked = active); process leg proxied, **setup taken from the clean machine registration**; setup reconciles to the minute as the built-in control.
- **Impact on Slide 8:** because RCCP consumes these route standards, the capacity outlook carries this known error until routes are aligned.
- **Journey:** phase 1 = labour-union (this cycle); phase 2 = OFS true runtime + OEE and calendar-based break rule. Show the gap trending down over cycles as routes align.
- *Priority: cost + availability | Parameter: Adapt (documented vulnerability) | Source: D365 T&A + ProdRouteJob; method in Validated Findings register*

### 10 — Planning rates — review & recommendations (management item 5)
- **Message:** are the rates we plan and cost against right.
- **July: placeholder / light.** Early callouts only — e.g. evidence that configured rates diverge from realised (ties directly to Slide 9; OFS Performance >100% on some jobs indicates ideal rate ≠ route standard).
- Recommendations / what develops next cycle.
- *Priority: cost | Parameter: Measure + Adapt | Source: D365 route rates vs realised; OFS ideal-rate reconciliation (phase 2)*

### 11 — Assumptions & vulnerabilities (OW-required)
- **Message:** the documented basis of the forward plan, so it can be revalidated if things change (p5).
- Register-style list: route-standard accuracy (Slide 9), RCCP status (Slide 8), SOH policy-target gaps (Slide 6), data-maturity caveats, commodity / FX exposure (if any).
- *Priority: all | Parameter: Adapt | Source: Validated Findings register + this deck's flags*

### 12 — Gaps, risks & actions — decisions requested
- **Message:** what the room needs to decide or own to close the gaps.
- Action log: item · owner · date · decision requested. Pull the flagged items from slides 6, 8, 9, 10.
- Clearly mark which need a **decision from the required audience** vs FYI.
- *Priority: all | Parameter: Adapt | Source: this deck*

### 13 — Roadmap: how the SR evolves
- **Message:** this is cycle one; here's the maturation path.
- Financial results integrated on Braulio's return · route-integrity phase 2 (OFS + calendar) · RCCP stand-up feeding the capacity outlook · SOH policy targets defined · planning-rate review built out.
- *Priority: — | Parameter: Align (to strategy over the horizon)*

---

## Build notes

- **13 slides.** Trim candidates if a tighter first cycle is wanted: merge 5+6 (inventory + SOH) and fold 10 into 7. Minimum viable = ~9 slides.
- **Data readiness per slide** (build order):
  - Ready now: 3, 4 (D365 header), 9 (validated on one order — pending population run).
  - Partial / provisional: 5, 6 (needs inventory source + policy targets), 8 (RCCP parked).
  - Placeholder for July: 10.
- **Openway theme:** match the existing Status Update deck — colours, layout, word count. One message per slide; KPI-led.
- **Methodology traceability:** every content slide tagged to a parameter and a priority above, so the build is demonstrably OW-consistent, not ad hoc.
