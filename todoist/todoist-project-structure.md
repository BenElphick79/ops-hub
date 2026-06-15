# Todoist Work Project Structure

A reference for what belongs where. When a new task lands, use this to decide which project it goes in — don't overthink it, imperfect placement is better than inbox rot.

---

## Projects

### 📅 Production Scheduling
**Scope:** Everything related to the day-to-day and forward scheduling of production across all lines. This is operational execution.

**Belongs here:**
- Gantt chart actions and updates
- Batch order firming and rescheduling
- Capacity constraint decisions and escalations
- Started/ended order discrepancies needing follow-up
- Resource-specific scheduling issues (use resource labels to tag which line)
- Forward schedule reviews and adjustments
- Any D365 scheduling behaviour that needs investigation or a workaround

**Does not belong here:**
- MRP configuration changes → Supply Planning / MRP
- Structural process improvements → Projects & Initiatives
- Comms to leadership about scheduling → Stakeholder & Comms

---

### 🔧 Supply Planning / MRP
**Scope:** The planning engine, its configuration, and the outputs it produces. Everything upstream of execution.

**Belongs here:**
- Action message review and response
- Coverage group configuration tasks
- Safety stock reviews and changes
- Negative days / receipt margin investigations
- Pegging anomaly investigations
- Forecast consumption issues (e.g. reduction key behaviour)
- MRP discovery findings that need actioning
- Item-level coverage record gaps
- FEFO, period bucket, or fence-related planning issues

**Does not belong here:**
- Executing what MRP tells you to do (firming orders) → Production Scheduling
- Presenting MRP findings to leadership → Stakeholder & Comms

---

### 🏗️ Projects & Initiatives
**Scope:** Structural, one-off work that improves the function rather than running it. Things with a defined start and end.

**Belongs here:**
- S&OP design and setup work
- Two-plan architecture (Static + RCCP) development
- MRP discovery project tasks
- New resource configuration (onboarding a new line, new coverage group design)
- Process documentation and work instructions
- Anything that would appear on a project plan or Gantt

**Does not belong here:**
- Recurring operational tasks → Production Scheduling or Supply Planning / MRP
- Exec updates on project status → Stakeholder & Comms

---

### 📣 Stakeholder & Comms
**Scope:** Anything that involves another person's input, approval, awareness, or response. Tasks that are blocked on someone else or need to go to someone.

**Belongs here:**
- Executive and management updates to draft or send
- Escalations waiting on a decision
- Follow-ups on unanswered questions
- Meeting prep tasks
- Anything where the next action is "talk to [person]" or "wait for [person]"
- Supply constraint communications (e.g. Honey Graham-style shortage updates)

**Does not belong here:**
- The underlying supply or scheduling issue itself → relevant operational project
- Internal analysis work → Supply Planning / MRP or Projects & Initiatives

---

### ⚙️ Admin
**Scope:** Everything that doesn't fit above. The catch-all for low-friction work that still needs to be tracked.

**Belongs here:**
- System access requests
- Training and certification tasks (MB-330, MB-335)
- Expense claims, HR tasks, onboarding paperwork
- Non-urgent tool setup (e.g. Todoist itself, new apps)
- Anything miscellaneous that has a due date but no strategic weight

**Does not belong here:**
- Anything that's actually operational — be honest with yourself and file it properly

---

## Labels (Resource Tags)

Apply these to individual tasks across any project to enable filtering by production line.

| Label | Resource |
|-------|----------|
| `#RCH` | RCH line |
| `#G01` | G01 line |
| `#Y04` | Y04 line |
| `#Y05` | Y05 line |

> Add remaining production pool labels as needed. The goal is to be able to filter "all open tasks touching Y04" across every project at once.

---

## Priority Levels

Use these consistently. A short, honest P1 list is the most valuable thing in this system.

| Priority | Meaning |
|----------|---------|
| **P1** | On fire — needs action today |
| **P2** | This week, no later |
| **P3** | On the radar — has a due date but not urgent |
| **P4** | Someday / reference / low weight |

---

## Rules of the Road

- **Process your Inbox daily.** Anything sitting in Inbox is unmanaged. Move it to a project or delete it.
- **P1 should be short.** If everything is P1, nothing is. Be ruthless.
- **Resource labels are optional but useful.** Not every task needs one — only tag when the resource context matters for filtering.
- **Imperfect placement beats no placement.** If you're not sure which project a task belongs in, pick the closest one and move on.
