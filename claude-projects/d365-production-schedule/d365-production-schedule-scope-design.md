# D365 Production Schedule — Project Scope Design

**Status:** STRAWMAN v0.1 — draft for iteration, not a locked spec
**Owner:** Ben (Supply Planning Manager)
**Entity in scope:** AXF (manufacturing / supply)
**MRP engine:** Planning Optimization
**Last updated:** 2026-07-11

---

## 1. Purpose

Automate the export of the current on-system production schedule from D365 to Excel for weekly distribution across the business.

The manual, offline duplicate of the schedule is being retired. That copy is out of date the moment it is issued and costs re-keying time each cycle. This project deletes that manual step, not by improving the offline copy, but by regenerating it on demand from the system of record.

Secondary opportunity (later phase): a private validation layer that flags on-system scheduling gaps for correction before distribution — a safety net for the owner, not a distributed artefact.

---

## 2. Directive & honest framing

This is a **current business directive**, not the owner's assessment of best practice.

The owner's position, recorded here so it is not lost as the project matures:

- The schedule lives on the system and should **only** live on the system.
- D365 should be the **single source of truth** and single point of reference for all parties.
- A distributed "in writing" copy converts a live, correctable system state into a frozen artefact that gets forwarded, screenshotted, and quoted back weeks later.

The project proceeds under the directive while holding these principles. The design choices below exist to contain the downsides of distribution, not to endorse it.

---

## 3. Design tenets

1. **System is source of truth.** The export is a disposable, regenerable snapshot — never a parallel schedule to be maintained by hand.
2. **The export is stateless.** If it is lost, it is regenerated in one refresh. No manual edits survive between refreshes.
3. **Validation outputs are private.** Seen only by the owner, used only to trigger corrections *in D365*, never distributed. The moment a validation output is shared, it stops being a safety net and becomes another artefact the owner is accountable for.
4. **Every issued schedule carries the caveat** (Section 4). This is the cheapest and highest-leverage control in the whole project.
5. **Recurring flags are a config to-do list, not a permanent manual task.** If a check keeps firing for the same error class, the fix is upstream (scheduling params, firming discipline, route/BOM config), and the goal is to make the check go quiet — not to catch it by hand forever.

---

## 4. Distribution caveat (mandatory, travels with every issue)

To be included on the distribution table / cover of every issued schedule:

> **This schedule is a point-in-time snapshot exported from D365 on [refresh date/time]. D365 is the single source of truth for live production status — refer to the system for current status. This copy is subject to change and will be out of date as soon as the schedule changes.**

Rationale: this reframes every future "but the schedule said X" from an owner error into a stale-copy problem — which is the honest framing anyway. It also hedges the tension of distributing "in writing" while the primary scheduling process is still stabilising.

> **Multi-sheet note (Phase 3):** the Phase 3 distribution workbook is split into
> separate stage sheets (see §5 / §7). Each sheet is separately forwardable, so
> the caveat must travel on **every distributed sheet**, not only a cover tab.

---

## 5. Phasing

| Phase | Name | Deliverable | Definition of Done (exit test) |
|---|---|---|---|
| **1** | Scope design | This document | Strawman agreed; open decisions logged. Stays open to ongoing development. |
| **2** | MVP build | Refresh → raw output table(s) | A schedule that refreshes and outputs table(s) in the expected **raw** structure, fields and content. **No validations. No flags. No shiny.** |
| **3** | Validation design & build | Raw table + flag tables + final distribution table | A schedule that refreshes and outputs the expected **final** structure with all validations applied, split into stage sheets for distribution. |

### Phase 2 guardrail (on record, at owner's request)
Phase 2 is raw output only. No validation logic, no flags, no tweaks, no "while I'm in here" additions. **This guardrail is to be enforced even against the owner's own scope creep during Phase 2.**

Note: the `Production Type` material-stage column (added to DIM_ProductionPool in
Phase 2) is **not** a guardrail breach — it is the coarse echelon indicator §7
already requires in Phase 2 (see §7). What is deferred to Phase 3 is any *use* of
that column to split, format, or shape the distribution output.

### Phase 3 output architecture
Phase 3 is expected to comprise:
1. The Phase 2 **raw output** table(s).
2. **Validation flag table(s)** — each holds *only* the rows that trip a given check.
3. The **final distribution table** — clean, formatted, caveated — **split into
   stage sheets** using the `Production Type` column: **Finished Goods (FG)**,
   **Intermediate/Batch (BATCH)**, **Pre-Weigh (PW)**. Most consumers read only
   the FG sheet (finished-good timing); PW/BATCH are of interest to a narrower
   audience. Once the stage column rides on the raw output, each sheet is a
   one-line filter — no new build is needed in Phase 2 to enable this.

**Confidence gate:** when every validation flag table is empty, the owner can be confident (as far as automated checks allow) that the final distribution table is clean. "All tables clear" is the go/no-go signal for issue.

---

## 6. Architecture (strawman)

```
D365 F&SCM
   │  (direct connection, daily refresh)
   ▼
Power BI dataset  ── single upstream source of truth for the export
   │  (connected in-workbook table -> Power Query; D1 CLOSED, see build doc)
   ▼
Excel (Power Query ETL)
   │
   ├─ Raw output table(s)            [Phase 2]   (carries Production Type stage tag)
   ├─ Validation flag table(s)       [Phase 3]
   └─ Final distribution table       [Phase 3]   (split into FG / BATCH / PW sheets)
```

**Cadence:** issue **weekly**; retain the ability to refresh **daily** for validation runs at the owner's discretion.

---

## 7. Raw output field set — STRAWMAN, confirm/correct

**Do not treat this as final.** This is a proposed skeleton to react to. Confirm each field against the actual Power BI dataset; add/remove as needed. Fields are grouped by role.

**Identity**
- Production order number
- Item number
- Item name / description
- Entity (AXF)
- Site / warehouse

**Placement (where it runs)**
- Resource / work centre — one of: CB2, CB3, CB4, CM2, CM3, CM4, G01, G02, G03, G08, G09, Y04, Y05
- Production Pool
- Production Area
- **Production Type (material stage)** — FG / BATCH / PW, one label per pool via
  DIM_ProductionPool. Added Phase 2. Doubles as the §7 echelon indicator (below)
  and as the Phase 3 stage-split key (§5). See build doc F6.

**Timing**
- Scheduled start (date/time)
- Scheduled end (date/time)
- Delivery / requirement date (need-by)
- Job sequence on resource

**Quantity**
- Order quantity
- Unit

**Status**
- Production order status (e.g. Created / Estimated / Scheduled / Released / Started / Reported as finished)

**Linkage — enables Phase 3 checks**
- **BOM level / echelon indicator** — **SATISFIED in Phase 2** by the
  `Production Type` stage column (coarse level: PW → BATCH → FG). This is a coarse
  echelon marker, not full pegging.
- **Parent / reference order (linkage across multi-resource, multi-echelon
  productions)** — **DEFERRED to Phase 3.** At batch-order-header grain there is
  no direct parent field; this linkage lives in reservations/pegging (build doc
  F2). It is therefore not present in the Phase 2 raw output, and the Phase 3
  "missing job in a multi-echelon production" check depends on reservation data
  not currently in scope.

> ⚠️ **Design note (reconciled 2026-07-11):** the earlier requirement to carry
> *both* linkage fields in Phase 2 has been split. The coarse echelon indicator
> is met now (via Production Type); the parent/reference-order linkage is a Phase 3
> reservations job (F2) and does not block Phase 2 sign-off. Date-based Phase 3
> checks (past-due, delivery-vs-scheduled) are unaffected — viable at header grain.
> **Split accepted by owner 2026-07-11.**

---

## 8. Open decisions & assumptions register

Nothing below is assumed as fact — logged for the owner to confirm.

| ID | Item | Status / question |
|---|---|---|
| **D1** | Excel ↔ Power BI dataset connection mechanism | **CLOSED.** Connected in-workbook table → Power Query via `Excel.CurrentWorkbook`. XMLA/Analysis Services route assessed and closed as not viable in this environment. See build doc F1. |
| **D2** | Raw field set (§7) | Proposed strawman only. Confirm against actual dataset fields. |
| **D3** | Dataset connection type (D365 → Power BI) | Stated as "direct connection, daily refreshed." Confirm whether DirectQuery or scheduled Import, as it affects freshness and refresh behaviour. Non-blocking. |
| **D4** | Order scope | **Split by branch. CLOSED.** STG applies active-status scope (Created, Estimated, Scheduled, Released, Started). Published (FCT) additionally issues firmed only — excludes Created + Estimated (D4b, resolved 2026-07-11; build doc F8). Validation branch retains all active statuses as a safeguard. |
| **D5** | Distribution format & channel | Excel file emailed? SharePoint? Confirm at Phase 2. Affects how the caveat is surfaced — note it must travel on each stage sheet (§4). |
| **D6** | Validation classes | Design in Phase 3 (see §9). Not built before then. |
| **D7** | Stage label register | `Production Type` values FG / BATCH / PW are floor vernacular on a material-stage axis (build doc F7). Revisit friendly display labels for the general audience when the Phase 3 distribution presentation is designed. Non-blocking. |

---

## 9. Validation classes — Phase 3 register (design only, NOT built yet)

Placeholders captured from the owner's intent. Each becomes a flag table in Phase 3. Purpose: private trigger to correct on-system gaps missed in the primary scheduling process.

- **Job overlap** — do two jobs overlap on the same resource?
- **Delivery vs scheduled date** — is the delivery/need-by date within *X* days of the scheduled date? (X to be defined.)
- **Missing job — multi-resource / multi-echelon** — is an expected job absent across a linked, multi-level production? (Depends on the parent/reference-order linkage deferred in §7 — i.e. on reservation/pegging data not yet in scope.)
- **Sequence validation** — is the job sequence on a resource valid / gapped?
- **Past-due** — scheduled or delivery date already in the past relative to refresh.

Each check should, where it recurs, feed back into upstream config remediation (design tenet 5).

---

## 10. Explicitly out of scope (crawl-walk-run)

- Any offline schedule that is manually edited and maintained.
- Distributing validation outputs to anyone but the owner.
- Write-back to D365 from Excel.
- Automated corrections (checks flag; the human corrects in-system).
- Anything shiny in Phase 2 — including *using* the Production Type column to
  split or format output (that is the Phase 3 distribution sheet, not Phase 2).
