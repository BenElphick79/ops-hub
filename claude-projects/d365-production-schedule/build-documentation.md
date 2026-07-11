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
SRC_BatchOrderHeader  ->  STG_ProductionSchedule
   │                        shared base: pool + status scope, stage tag,
   │                        date split, renamed, sorted — NO window filter
   ├─ FCT_ProductionSchedule            [Phase 2, published]
   │     + next-week window filter
   │     + PENDING: exclude Created / Estimated (firmed only)  ← D4b, still open
   └─ STG_ProductionScheduleValidation  [Phase 3 stub]
         forks upstream of the window filter (sees all time, all statuses)
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

**Query naming.** The staging / published / validation queries were renamed
`*BatchOrderHeader` → `*ProductionSchedule` (STG, FCT, validation). `SRC_` keeps
the `BatchOrderHeader` name (it maps 1:1 to the source table). Names in this
document track the build.

---

## 2. Query lineage (current)

- **PAR_DateRanges** — parameter record. Computes `NextWeekStart` (Monday of next
  week) and `NextWeekEnd` (following Sunday) from today. Cleaned: no longer emits
  `Granularity`; header no longer references the deleted `FNC_GenerateDateTable`.
- **DIM_ProductionPool** — hardcoded in-scope pool dimension, **now two columns**:
  `Production Pools` (key) and `Production Type` (material-stage tag). Enter-Data
  literal (`Table.FromRows` over a compressed row set). **16 pools** (was 17 — the
  combined CB2&CB3 batch-only pool was removed as redundant; see F6). Stage values:
  FG / BATCH / PW.
- **DIM_OrderStatus** — hardcoded active-status list: Created, Estimated,
  Scheduled, Released, Started (finished/closed excluded). Enter-Data literal.
- **SRC_BatchOrderHeader** — reads `Table_BatchOrderHeader`; strips bracketed
  column names; selects the 9 in-scope columns; sets types (`SumScheduledQuantity`
  typed as `type number` / decimal).
- **STG_ProductionSchedule** — shared base for both forks. Inner-joins
  `DIM_ProductionPool` (in-scope-pool filter) and **expands `Production Type`**;
  inner-joins `DIM_OrderStatus` (active-status filter, expanded nothing — filter
  only); splits scheduled start/end into date + time via a typed record +
  `ExpandRecordColumn`; selects the final column set via a `SelectColumns`
  whitelist (which carries `Production Type` and positively drops the leftover
  `DIM_OrderStatus` nested column and the datetime sources); renames to
  presentation names; sorts by pool, then start date, then start time.
  **No window filter** — that lives on the published branch.
- **FCT_ProductionSchedule** — published (Phase 2) branch. Source = STG, then the
  next-week window overlap filter (keeps jobs active at any point in the window,
  so multi-week spanning jobs are retained). **Pending (D4b):** additionally
  restrict to firmed/scheduled statuses (drop Created / Estimated) so only firmed
  work is issued. Not yet implemented as of the 2026-07-11 review (v1.3).
- **STG_ProductionScheduleValidation** — Phase 3 stub. Source = STG (pass-through),
  forking upstream of the window filter, so it retains Created / Estimated and is
  not constrained to next-week. Currently time-unbounded (see F5).

**Grain:** batch-order-header — one row per batch order. Confirmed appropriate
for Phase 2; matches current practice.

**Output column set (STG, presentation names):** Production Type, Production Pool,
Batch Order Number, Item Number, Item Name, Delivery Date, Batch Order Status,
Scheduled Quantity, Scheduled Start Date, Scheduled Start Time, Scheduled End
Date, Scheduled End Time.

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
The FCT exclusion is NOT yet implemented — see Open Items (D4b). Same whitelist
pattern as pools, so the same silent-drop on any unlisted status value.

F5 — Fork architecture: window filter moved to the published branch.
  [2026-07-10]
The next-week window overlap filter was moved out of STG into FCT (published).
STG is now the shared, unfiltered base; the validation fork sits upstream of
the window filter, so it is not boxed into next-week. Consequence: validation
currently draws off a time-unbounded set — Phase 3 will want its own broader
window (rolling / not-yet-finished), not zero window. Also: the DIM_OrderStatus
leftover nested-join column is dropped via the STG SelectColumns whitelist; and
the date-spine infra (FNC_GenerateDateTable, DIM_Datetable) was removed as
ahead-of-phase — may return in Phase 3.

F6 — DIM_ProductionPool second column: material-stage tag. [2026-07-11]
DIM_ProductionPool gains [Production Type], a coarse material-stage code per
pool: FG (finished goods), BATCH (intermediate), PW (pre-weigh). Grain = pool;
one stage per pool, holds by construction (single value per row). Serves the
§7 coarse echelon indicator required in Phase 2; does NOT discharge the Phase 3
reservation/pegging multi-echelon linkage — F2 stands. The column reaches the
raw output via the STG DIM-join expand (Production Type) + the SelectColumns
whitelist; it is the leading output column and is not renamed. In-scope pools
now 16 (was 17): the combined CB2&CB3 batch-only pool was removed as redundant,
its orders present under CB2/CB3 individually — confirmed nothing hidden. G03
tagged FG (corrected; previously mis-scoped as intermediate). Embedded 16-row
Enter-Data table verified against the agreed map on 2026-07-11. Intended
downstream use — a three-sheet split of the distribution workbook by stage
(FG / BATCH / PW) — is Phase 3 (final distribution table), not built in Phase 2.

F7 — Label register: FG / BATCH / PW are floor vernacular on a stage axis.
  [2026-07-11]
The [Production Type] values and the column axis were confirmed as material
stage, but the value codes (FG/BATCH/PW) are shop-floor terms. "BATCH" as a
stage value is a process word standing in for "intermediate", and the whole
table is batch orders, so it invites conflation for a general reader. Kept
deliberately for now (owner's call). Flagged for a friendly-display-label
revisit when the Phase 3 distribution presentation is designed — see Open Items.
```

---

## 4. Open items

```
BLOCKS PHASE 2 COMPLETION
- D4b — FCT (published) must exclude Created + Estimated (issue firmed /
  scheduled only). STILL NOT IMPLEMENTED as of the 2026-07-11 review (v1.3):
  FCT_ProductionSchedule applies the next-week window filter only. Fix is a
  firmed-status whitelist appended to FCT:
      FilteredRowsFirmedOnly = Table.SelectRows(
          FilteredRowsNextWeekOverlap,
          each List.Contains({"Scheduled","Released","Started"},
              [Batch Order Status]))
  Requires confirmation it is resolved before Phase 2 sign-off.

- §7 linkage — parent/reference order half not met (conscious, per F2). The
  Production Type stage column satisfies the §7 coarse echelon indicator. The
  parent/reference-order half is not in the raw output because it lives in
  reservations/pegging (F2), which is Phase 3. Scope §7 wording amended to
  reflect this split so Phase 2 sign-off is not blocked by an un-buildable
  requirement. Confirm the amended §7 is accepted at sign-off.

NON-BLOCKING
- D3 — D365 -> dataset leg: Import vs DirectQuery unconfirmed (freshness only).
- F7 label register — revisit friendly display labels for FG/BATCH/PW (and the
  column name) when the Phase 3 distribution presentation is designed. "BATCH"
  will not read as "intermediate" to a general audience.
- Finalisation sweep — PAR_DateRanges dangling refs now CLEARED. Remaining:
  most queries still lack the mandatory 4-line header block, and step names are
  Power Query defaults (ChangedType, RemovedOtherColumns, etc.). Header +
  step-name-prefix standardisation deferred to the Claude Code finalisation
  pass on GitHub upload.
- DIM key naming — DIM_ProductionPool key column is "Production Pools" (plural)
  vs the output's "Production Pool" (singular). Harmless (never collide); tidy
  in the finalisation pass if desired.
```
