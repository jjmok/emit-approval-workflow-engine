# Project Decisions Log

_Last updated: 2026-07-06_

This file records decisions made during planning discussions that are **not**
captured in the original source documents (the spec, the process explainer, or the
client Q&A), or that **override** them. Where this file conflicts with an older source
document, **this file wins** — it reflects the latest agreed position.

Each decision notes what was decided, why, and its status.

---

## D-1. Business-hours as the base timing unit

**Decision:** All SLA and KPI timing is measured in **business hours** as the base
unit. A business day is a derived multiple of the configured working hours (e.g.
8 business hours = 1 day, 4 = half a day). Day- and half-day intervals are computed
from hours, never stored as the base unit. Wall-clock time is never used for SLA/KPI
measurement.

**Applies to which clocks:** By default the working calendar governs **both** the
per-person response SLA clock **and** the step KPI clock, so no one is penalised for
time the office was closed. The calendar is configurable per clock-type, so a
different calendar could later be applied to the KPI clock versus the personal clock
without a code change.

**Note on client input:** The client's Q&A answer said business-time is "only for the
KPI's." We are applying it to both clocks by default because running personal
deadlines on raw wall-clock time would auto-escalate people over nights/weekends
through no fault of their own — contradicting the system's own "nobody penalised for
closed-office time" promise. The per-clock configurability preserves the client's
option to change this later without a rebuild.

**Open confirmation:** Confirm with client that personal (per-person) response
deadlines should also pause outside working hours, not just the KPI clock.

**Status:** Decided (base unit + both-clocks default). One confirmation outstanding.

**Reflected in:** Constitution v1.3.x — Technology & Platform Constraints
(business-hours timing + shared/per-clock working calendar).

---

## D-2. Edit / resubmit restarts the flow from zero

**Decision:** When a task's **request details are actually changed** (edited and
resubmitted), the task **restarts from step 1** — the first approver again — and all
prior approvals are **discarded**. Approval always applies to the exact version that
was approved; once the content changes, earlier approvals are no longer valid.

**Boundary — what does NOT restart:** A plain **request-for-information / clarification**
where the request details do **not** change is **not** an edit. In that case the task
**resumes in place** (same step, same assignee) per the spec's info-request behaviour.
The restart-from-zero rule applies only when the underlying request is modified.

**Source:** Client Q&A — "its locked, only can add notes, if clarifications are
required by an approver, any changes would require re-approval, start from zero" — and
confirmed in discussion ("start from zero, the 1st approver again").

**Conflicts with:** The original spec's resume-in-place model (FR-21 boss-override
resumes at the same step). That resume-in-place behaviour still applies to the
**boss-review / override** path after a rejection; D-2 governs the separate case of the
**requester editing and resubmitting** the request content. These are two different
triggers — keep them distinct in the spec.

**Status:** Decided.

**To be formalised in:** `/speckit.specify` (this is a spec-level rule, deliberately
kept out of the constitution).

---

## D-3. Requester cannot approve their own task (segregation of duties)

**Decision:** The user who raises a task can **never** be an approver of their own task
at any step, enforced server-side, across every flow.

**Clarification:** Receiving a task back to **revise and resubmit** (after a rejection
is upheld) is **not** approving it — the raiser may revise their own returned task;
they simply may not act as an approver of it.

**Source:** Client Q&A — "any process should always be an approver to the next step."

**Status:** Decided.

**Reflected in:** Constitution v1.3.x — Principle X (Segregation of Duties).

---

## D-4. Cancellation before approval

**Decision:** A task may be **cancelled at any stage before it is approved**, and the
cancellation is **recorded** in the history/audit trail. Cancellation is a **status
change, not a hard delete** — the record is retained.

**Source:** Client Q&A — "Yes, at any stage, it can be cancelled."

**Status:** Decided.

**To be formalised in:** `/speckit.specify`. (No-hard-delete is already in the
constitution's Code Quality & Data Standards.)

---

## D-5. Task creation is restricted to designated users

**Decision:** Raising a task is **not** open to every user. Task creation is
restricted to specific, **designated** users.

**Source:** Proposal email suggestion, aligned by client's platform framing.

**Status:** Decided (pending final confirmation of who the designated creators are per
task type — a configuration value, not logic).

**To be formalised in:** `/speckit.specify` + configuration.

---

## D-6. Details locked after submission; notes only

**Decision:** Once a task is submitted, its details are **locked**. Users may **add
notes** but not alter the submitted request in place. Any actual change to the request
goes through the edit/resubmit path, which triggers restart-from-zero (see D-2).

**Source:** Client Q&A — "its locked, only can add notes."

**Status:** Decided.

**To be formalised in:** `/speckit.specify`.

---

## Open questions (not yet decided — do NOT assume)

These are recorded so they are not silently resolved by assumption. They are **not**
decisions yet.

- **OQ-1. Email-based approval.** Client asked whether approval can be done from the
  notification email itself. Not yet decided. The system is mobile-push-first; email
  approval would be an additional capability. Flag to client before speccing.

- **OQ-2. Business-time on personal clock.** See D-1 — confirm the personal response
  clock also pauses outside working hours (we have defaulted it to yes).

- **OQ-3. Hosting specifics.** Self-hosted is decided (Constitution Principle XII);
  the specific host/infrastructure is a deployment-config decision still to be named.

- **OQ-4. Top escalation authority.** The single company-wide top escalation person
  (FR-25a) must be named — a configuration value, not logic.

---

## How to use this file

- When `/speckit.specify` runs, treat this file as authoritative for the decisions
  above, **over** any conflicting statement in the older source documents.
- When an open question (OQ-*) is resolved, move it up into a numbered decision (D-*)
  with its rationale and status.
- Keep this file in `docs/source/` alongside the original documents so the agent reads
  it together with them.
