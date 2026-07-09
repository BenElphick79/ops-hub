# Validated Findings Register — AXF / OPEN D365 F&SCM

Ground truth for confirmed system behaviour. Append-only. One line of context, one outcome.
Status values: **VALIDATED** (confirmed in-system) · **DIAGNOSED** (root cause identified, fix not yet confirmed) · **RECOMMENDED** (advice given, not yet tested)

> Setup task: review the statuses below and correct any that have since been validated or disproven. After that, entries are added only when something is confirmed in-system.

---

## MRP / Planning Optimization

- **[2026-07-08] — Blanket receipt margin (7 days, all products)** — Confirmed as a source of false MRP shortage signals. *Status: VALIDATED.* Remediation: item coverage overrides on key Oats raw materials as test scope — planned, not yet started. *Fix status: RECOMMENDED — pending test results.*
- **[2026-07-08] — Finite capacity time fence = 90 days on Static plan** — Caused planned orders to schedule forward rather than backward from requirement date (vs 365-day coverage horizon). *Status: VALIDATED (behaviour confirmed).* Remediation: plan-level capacity time fence override enabled and set to 0, neutralising the 90-day fence — finite capacity effectively off for master planning while Finite capacity remains Yes. Change made 2026-07-08. *Fix status: VALIDATED — regeneration confirmed no finite capacity applied to planned orders; backward scheduling restored. Side-check closed 2026-07-08: no resource-specific finite capacity configs remain on RCH or any other resource — all resources fall under the plan-level capacity fence = 0.*
- **[2026-07-08] — Marking created by PlanOpt on planned order firming** — Fixed via Update marking = No at master planning parameters level; confirmed no MRP marking is being applied on firming. *Status: VALIDATED.* Known exception (expected behaviour): production orders created directly from a sales order still create marking — that path uses order-to-order marking and operates outside MRP, unaffected by the parameter.
- **[2026-07-08] — Multi-level BOM firming** — Confirmed in-system: planned order firming is level-by-level with no cascade automation available in Planning Optimization. Each level must be firmed separately. *Status: VALIDATED (system limitation / expected behaviour).*
- **[2026-07-08] — RCCP master plan** — Intended as a dedicated plan (infinite capacity, 365-day horizon) for capacity/S&OP visibility, running operation scheduling only. *Status: NOT RESOLVED — parked.* Blocker found on initial attempt: could not get operation-scheduling-only behaviour because the 90-day capacity time fence still job-scheduled everything within that window. *(Note the relationship to the fence entry above — the plan-level fence = 0 change may have altered this behaviour; worth re-testing as the starting point.)* **PRIORITY: #1 — capacity charts needed within ~1 week. Tracked as a separate project, not in this register.**
- **[2026-07-08] — Rolling 30-day reduction key (OPEN)** — Identified as cause of silent forecast consumption drops. *Status: DIAGNOSED — open, no remediation work started yet.*

## Production scheduling / capacity

- **[historical] — Capacity planning = Production set to No** (master planning parameters) — Gap found during RCH finite scheduling stand-up. *Status: CLOSED 2026-07-08 — superseded; finite capacity deliberately not in use, all resources governed by plan-level capacity fence = 0 (see fence entry above).*
- **[historical] — Expired calendar records on RCH** — Identified and resolved during finite scheduling configuration. *Status: CLOSED 2026-07-08 — calendars fixed; RCH remains an active resource. The finite scheduling configs from that stand-up are retired. Retained for reference if finite scheduling is reintroduced.*
- **[date TBC] — ProdSupervisorScheduledOrders view error** — Root cause: invalid Pool field in advanced filter. *Status: pending confirmation the fix held.*
- **[2026-07-08] — Machine (Packout) actual run-time not captured in D365; only start/finish** — Packout machine registers a single open job span per order, so D365 route/T&A "Hours" = elapsed span incl. idle/overnight/weekend, not active run-time. Confirmed on BO013235: 171.71h fed-back vs 65.36h standard; all T&A Calculated…Seconds fields = end−start to the second. Labour leg unaffected (per-shift clocking) and reconciles sensibly: 236.90h actual vs 326.80h standard (~28% under). *Status: VALIDATED — systemic for Packout, confirmed by SPM.* Remediation is phased — **Phase 1:** labour-union proxy for active line time (see next entry), adopted for the July Supply Review. **Phase 2 (parked):** OFS for true machine run-time + OEE (mapped via SKU→Item, Line→Resource); and calendar working-time intervals for a defined intra-shift break rule. D365 estimate/route standard retained as the planned leg throughout. *Fix status: Phase 1 RECOMMENDED — population validation pending; Phase 2 parked.*
- **[2026-07-08] — Labour-union proxy for active line time (Phase 1 machine-actual workaround)** — Where machine process time is uncapturable, derive active line hours from labour clock data: the line is active whenever ≥1 labour resource is clocked on; take the UNION of labour intervals — never the sum (overlapping workers overcount: 236.9h sum vs 59.8h union on BO013235). Split by RouteJobType — setup actual from the machine registration (D365 captures setup cleanly, 0.47h); process actual = union of process-type labour intervals vs route standard. Validated on BO013235: process-union 59.35h vs 65.36h standard (−9.2%), against a contaminated machine span of 171h. Built-in control: on setup (the leg D365 captures correctly) machine and labour-union agree to the minute (0.47h = 0.47h), evidencing the proxy is sound where cross-checkable. Non-working periods self-exclude without calendar dependency — Mon 9 Mar (VIC Labour Day) correctly dropped, no labour clocked. Phase-1 break handling = simple stated tolerance (materiality <1h on sample). *Status: RECOMMENDED — validated on one order; population run (all FP0x resources, full month) pending before KPI use.* **Phase-2 refinements parked alongside OFS:** calendar working-time intervals (confirmed to carry intra-day breaks per line) for a run-through-vs-stop break rule; and OFS ideal-rate vs D365 route-standard reconciliation, to validate before OEE Performance is reported "vs standard".

## Warehouse / BOM sourcing

- **[2026-07-08] — Formula components flushing from OBS instead of OCS** — Root cause traced to warehouse fields on the resource record. *Status: CLOSED — not currently relevant; was a one-off requirement now owned by someone else. Root cause retained for reference should it recur.*

## Open threads (unresolved, not findings)

*None currently.*

---

## Entry template

- **[YYYY-MM-DD] — Finding** — One-line context. Outcome. *Status: VALIDATED.*
