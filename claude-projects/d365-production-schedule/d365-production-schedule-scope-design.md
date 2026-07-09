# D365 Production Schedule — Project Scope Design

**Status:** STRAWMAN v0.1 — draft for iteration, not a locked spec
**Owner:** Ben (Supply Planning Manager)
**Entity in scope:** AXF (manufacturing / supply)
**MRP engine:** Planning Optimization
**Last updated:** 2026-07-10

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

---

## 5. Phasing

| Phase | Name | Deliverable | Definition of Done (exit test) |
|---|---|---|---|
| **1** | Scope design | This document | Strawman agreed; open decisions logged. Stays open to ongoing development. |
| **2** | MVP build | Refresh → raw output table(s) | A schedule that refreshes and outputs table(s) in the expected **raw** structure, fields and content. **No validations. No flags. No shiny.** |
| **3** | Validation design & build | Raw table + flag tables + final distribution table | A schedule that refreshes and outputs the expected **final** structure with all validations applied. |

### Phase 2 guardrail (on record, at owner's request)
Phase 2 is raw output only. No validation logic, no flags, no tweaks, no "while I'm in here" additions. **This guardrail is to be enforced even against the owner's own scope creep during Phase 2.**

### Phase 3 output architecture
Phase 3 is expected to comprise:
1. The Phase 2 **raw output** table(s).
2. **Validation flag table(s)** — each holds *only* the rows that trip a given check.
3. The **final distribution table** — clean, formatted, caveated.

**Confidence gate:** when every validation flag table is empty, the owner can be confident (as far as automated checks allow) that the final distribution table is clean. "All tables clear" is the go/no-go signal for issue.

---

## 6. Architecture (strawman)

```
D365 F&SCM
   │  (direct connection, daily refresh)
   ▼
Power BI dataset  ── single upstream source of truth for the export
   │  (Power Query connection — mechanism TBD, see §8 open decision D1)
   ▼
Excel (Power Query ETL)
   │
   ├─ Raw output table(s)            [Phase 2]
   ├─ Validation flag table(s)       [Phase 3]
   └─ Final distribution table       [Phase 3]
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

**Linkage — REQUIRED IN PHASE 2 to enable Phase 3 checks**
- BOM level / echelon indicator
- Parent / reference order (linkage across multi-resource, multi-echelon productions)

> ⚠️ **Design dependency:** the linkage fields above must be present in the Phase 2 raw output even though Phase 2 builds no validation. Phase 3's "missing job in a multi-echelon production" check cannot function without them. Omitting them in Phase 2 forces a Phase 2 rework later.

---

## 8. Open decisions & assumptions register

Nothing below is assumed as fact — logged for the owner to confirm.

| ID | Item | Status / question |
|---|---|---|
| **D1** | Excel ↔ Power BI dataset connection mechanism | Not yet decided. Three viable routes, each with different transform freedom: **(a)** live connection to published semantic model — least M-transform freedom, PivotTable-style; **(b)** DAX query pulled via Power Query — good balance, shapeable, but you own the DAX; **(c)** export/flat table then transform — most M freedom, least "live." Resolve at Phase 2 kickoff; it constrains the entire ETL design. |
| **D2** | Raw field set (§7) | Proposed strawman only. Confirm against actual dataset fields. |
| **D3** | Dataset connection type (D365 → Power BI) | Stated as "direct connection, daily refreshed." Confirm whether DirectQuery or scheduled Import, as it affects freshness and refresh behaviour. |
| **D4** | Order scope | Which order states are in scope for distribution? Firmed/released only, or including planned? (Assumption: firmed production orders — confirm.) |
| **D5** | Distribution format & channel | Excel file emailed? SharePoint? Confirm at Phase 2. Affects how the caveat is surfaced. |
| **D6** | Validation classes | Design in Phase 3 (see §9). Not built before then. |

---

## 9. Validation classes — Phase 3 register (design only, NOT built yet)

Placeholders captured from the owner's intent. Each becomes a flag table in Phase 3. Purpose: private trigger to correct on-system gaps missed in the primary scheduling process.

- **Job overlap** — do two jobs overlap on the same resource?
- **Delivery vs scheduled date** — is the delivery/need-by date within *X* days of the scheduled date? (X to be defined.)
- **Missing job — multi-resource / multi-echelon** — is an expected job absent across a linked, multi-level production? (Depends on §7 linkage fields.)
- **Sequence validation** — is the job sequence on a resource valid / gapped?
- **Past-due** — scheduled or delivery date already in the past relative to refresh.

Each check should, where it recurs, feed back into upstream config remediation (design tenet 5).

---

## 10. Explicitly out of scope (crawl-walk-run)

- Any offline schedule that is manually edited and maintained.
- Distributing validation outputs to anyone but the owner.
- Write-back to D365 from Excel.
- Automated corrections (checks flag; the human corrects in-system).
- Anything shiny in Phase 2.
