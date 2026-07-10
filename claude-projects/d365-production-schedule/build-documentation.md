# D365 Production Schedule — Build Documentation

> Describes the schedule **as built**. The workbook and its Power Query are the
> ultimate source of truth; if this document and the build diverge, the build
> wins. Versioning is controlled in git; no version number in the filename.

---

## 1. Data source — LOCKED (D1 resolved)

Pipeline as built:

```
D365 F&SCM
   │  direct connection, daily refresh
   ▼
Power BI semantic model (dataset)
   │  in-workbook table connected to the dataset; column names arrive
   │  bracketed, e.g. [BatchOrderNumber]
   ▼
Excel workbook table: Table_BatchOrderHeader
   │  Excel.CurrentWorkbook(){[Name="Table_BatchOrderHeader"]}
   ▼
SRC_BatchOrderHeader  ->  STG_BatchOrderHeader
   │                        shared base: pool + status scope, date split,
   │                        renamed, sorted — NO window filter
   ├─ FCT_BatchOrderHeader              [Phase 2, published]
   │     + next-week window filter
   │     + PENDING: exclude Created / Estimated (firmed only)
   └─ STG_BatchOrderHeaderValidation    [Phase 3 stub]
         forks upstream of the window filter (sees all time)
```

**Mechanism.** Power Query consumes an in-workbook table that is connected to the
Power BI dataset, read via `Excel.CurrentWorkbook`. Power Query does **not**
connect to the semantic model directly.

**Why the intermediate table (D1 resolution).** In the current environment,
Power Query cannot consume the Power BI semantic model as a transformable (M)
source. The only direct-to-Power-Query route — the Analysis Services connector
against the model's XMLA endpoint — is **not a valid option in this environment**
and is not pursued. The connected-table → Power Query pattern is the locked
mechanism. **D1: CLOSED.**

**Adjacent, still open:** the D365 → dataset leg is stated as a direct
connection, daily refresh. Whether that leg is Import or DirectQuery (**D3**) is
not yet confirmed; it affects data freshness but not the build mechanism above.

---

## 2. Query lineage (current)

- **SRC_BatchOrderHeader** — reads `Table_BatchOrderHeader`; strips bracketed
  column names; selects the 9 in-scope columns; sets types (Scheduled Quantity
  typed as decimal / `type number`).
- **STG_BatchOrderHeader** — shared base for both forks. Inner-joins
  `DIM_ProductionPool` (in-scope-pool filter) and `DIM_OrderStatus` (active-status
  filter); splits scheduled start/end into date + time; selects the final column
  set via a `SelectColumns` whitelist (positively dropping the nested join columns
  and the datetime sources); renames to presentation names; sorts by pool then
  start. **No window filter** — that now lives on the published branch.
- **FCT_BatchOrderHeader** — published (Phase 2) branch. Source = STG, then the
  next-week window overlap filter (keeps jobs active at any point in the window,
  so multi-week spanning jobs are retained). **Pending:** additionally exclude
  Created / Estimated so only firmed/scheduled work is issued (see Open Items).
- **STG_BatchOrderHeaderValidation** — Phase 3 stub. Source = STG, forking
  upstream of the window filter, so it retains Created / Estimated and is not
  constrained to next-week. Currently time-unbounded (see F5).
- **PAR_DateRanges** — parameter record; supplies `NextWeekStart` /
  `NextWeekEnd`. `Granularity` is still emitted but unused, and the header still
  references the deleted `FNC_GenerateDateTable` — dangling, see Open Items.
- **DIM_ProductionPool** — hardcoded in-scope pool list (17 pools).
- **DIM_OrderStatus** — hardcoded active-status list: Created, Estimated,
  Scheduled, Released, Started (finished/closed excluded).

**Grain:** batch-order-header — one row per batch order. Confirmed appropriate
for Phase 2; matches current practice.

---

## 3. Findings & decisions log

```
F1 — D1 resolved (LOCKED): data source mechanism. [2026-07-10]
Power Query reads an in-workbook table (Table_BatchOrderHeader) connected to
the Power BI dataset, via Excel.CurrentWorkbook — not a direct connection to
the semantic model. The only direct-to-PQ route (Analysis Services against the
XMLA endpoint) is not a valid option in the current environment and is not
pursued — closed, not parked. D1 CLOSED on the connected-table pattern.

F2 — Grain confirmed header-level; echelon linkage lives in reservations.
  [2026-07-10]
Raw output is batch-order-header grain (one row/order); matches current
practice, accepted for Phase 2. Component/echelon links (CB2 in CB3, CM2 in
CM3; CB2/CM2/Y04/Y05 etc. drawing from G09) are not a direct parent field —
they surface only via reservations/pegging. Phase 3 "missing job in a
multi-echelon production" therefore depends on reservation data not currently
in scope, and is expected to be challenging. Parked to Phase 3. Date-based
checks (past-due, delivery-vs-scheduled) are unaffected — viable at header
grain.

F3 — Scheduled Quantity retyped Int64 -> decimal (type number) to avoid error
values on fractional process quantities. Confirmed in build. [2026-07-10]

F4 — D4 order-status scope: split by branch. [2026-07-10]
STG applies the shared active-status scope via DIM_OrderStatus inner join:
Created, Estimated, Scheduled, Released, Started (finished/closed excluded);
both forks inherit it. Published branch (FCT) must additionally exclude
Created + Estimated so only firmed/scheduled work is issued. Validation branch
retains Created + Estimated to flag "exists but not scheduled" as a safeguard.
The FCT exclusion is NOT yet implemented — see Open Items. Same whitelist
pattern as pools, so the same silent-drop on any unlisted status value.

F5 — Fork architecture: window filter moved to the published branch.
  [2026-07-10]
The next-week window overlap filter was moved out of STG into FCT (published).
STG is now the shared, unfiltered base; the validation fork sits upstream of
the window filter, so it is not boxed into next-week. Consequence: validation
currently draws off a time-unbounded set — Phase 3 will want its own broader
window (rolling / not-yet-finished), not zero window. Also this turn: the
DIM_OrderStatus leftover nested-join column was fixed by switching STG cleanup
to a SelectColumns whitelist; and the date-spine infra (FNC_GenerateDateTable,
DIM_Datetable) was removed as ahead-of-phase — may return in Phase 3.

(register candidate) DIM_ProductionPool inner join confirmed as an intentional
in-scope-pool filter, 17 pools, hardcoded. Residual: a genuinely new in-scope
pool drops silently until the dimension is updated — possible Phase 3
"unlisted pool" detector, not built.
```

---

## 4. Open items

```
BLOCKS PHASE 2 COMPLETION
- D4b — FCT (published) must exclude Created + Estimated (issue firmed /
  scheduled only). Not yet implemented. Requires confirmation it is resolved
  before Phase 2 sign-off.

NON-BLOCKING
- D3 — D365 -> dataset leg: Import vs DirectQuery unconfirmed (freshness only).
- Finalisation sweep — PAR_DateRanges still emits Granularity and its header
  references the deleted FNC_GenerateDateTable; clear both. Query four-line
  headers and step-name prefix standardisation are deferred to the Claude Code
  finalisation pass on GitHub upload.
```
