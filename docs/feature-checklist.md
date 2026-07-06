# EMIT Approval Workflow Engine — Capability Checklist

This document is a plain-English menu of what the EMIT Approval Workflow Engine can do
and where each capability stands in the build. It is meant for sharing and review — not
an engineering task list. Anything not listed here is **out of scope until it is added to
the specification first**, which keeps the spec the single agreed boundary of the system.

_Last updated: 2026-07-06_

## Status legend

| Symbol | Meaning |
|--------|---------|
| ⬜ | Not started |
| 🟡 | In progress |
| ✅ | Done |
| ⏸️ | Deferred |

> **Spec ref** is left blank where a specification item has not yet been assigned. These
> will be filled in as the spec formalises.

---

## 1. Task creation & templates

| Capability | Status | Spec ref | Notes |
|------------|--------|----------|-------|
| Raise a one-off task from a template | 🟡 In progress | BR-PA-1 | Slice 1: on-demand raise from a template |
| Schedule a recurring task that raises itself automatically | ⬜ Not started | | |
| Define the fields a template captures | ⬜ Not started | | |
| Restrict who is allowed to create each task type | ⬜ Not started | | |

## 2. Flow & routing

| Capability | Status | Spec ref | Notes |
|------------|--------|----------|-------|
| Route a task through a sequence of approval steps | 🟡 In progress | BR-PA-2, BR-PA-4, BR-PA-8 | Sequential steps; advance only on approval; one active holder |
| Configure the teams and the order they approve in | 🟡 In progress | BR-PA-2, BR-PA-3 | Steps as config (order, one team each); no code change |
| Support a single approving team or many teams in one flow | 🟡 In progress | BR-PA-2 | 1..N steps, one team or many |

## 3. Assignment & fallback

| Capability | Status | Spec ref | Notes |
|------------|--------|----------|-------|
| Assign to a primary person (PIC #1) | 🟡 In progress | BR-PA-3, BR-PA-7 | Single assignee per step this slice; PIC #2/boss fallback deferred |
| Fall back to a second person (PIC #2) if needed | ⬜ Not started | | |
| Fall back to the team boss as the final holder | ⬜ Not started | | |
| Give each person their own response window | ⬜ Not started | | |

## 4. Escalation — silence track

| Capability | Status | Spec ref | Notes |
|------------|--------|----------|-------|
| Move a task on when a person stays silent past their window | ⬜ Not started | | |
| Give each hop a fresh deadline | ⬜ Not started | | |
| Stop escalating once it reaches the team boss | ⬜ Not started | | |

## 5. Escalation — KPI track

| Capability | Status | Spec ref | Notes |
|------------|--------|----------|-------|
| Enforce a hard time cap on each step | ⬜ Not started | | |
| Escalate up the chain: boss → boss-of-boss → company top authority | ⬜ Not started | | |
| Use the higher of the two escalation clocks (max-wins ladder) | ⬜ Not started | | |
| Avoid raising the same escalation twice (deduplication) | ⬜ Not started | | |

## 6. Reviewer actions

| Capability | Status | Spec ref | Notes |
|------------|--------|----------|-------|
| Approve a task | 🟡 In progress | BR-PA-5, BR-PA-6, BR-PA-7, BR-PA-9 | Approve advances/closes; only the assignee may approve; requester may never approve own task |
| Reject a task with a mandatory reason | ⬜ Not started | | |
| Let a boss review a decision | ⬜ Not started | | |
| Choose whether an edit resumes or restarts the flow from zero | 🟡 In progress | BR-PA-10, BR-PA-11 | Decided (D-2/D-6): details locked, notes only; edit & resubmit restarts from step 1 |

## 7. Request for information

| Capability | Status | Spec ref | Notes |
|------------|--------|----------|-------|
| Pause the clocks while waiting for information | ⬜ Not started | | |
| Apply a response window to the creator who owes the information | ⬜ Not started | | |
| Escalate to the creator's boss if the information is not provided | ⬜ Not started | | |
| Withdraw a request for information | ⬜ Not started | | |

## 8. Delegation & cover

| Capability | Status | Spec ref | Notes |
|------------|--------|----------|-------|
| Forward a task to someone else on an ad-hoc basis | ⬜ Not started | | |
| Set up a standing delegation for leave | ⬜ Not started | | |
| Support a half-day delegation window | ⬜ Not started | | |

## 9. Cancellation

| Capability | Status | Spec ref | Notes |
|------------|--------|----------|-------|
| Cancel a task before it is approved | 🟡 In progress | BR-PA-12 | Cancellable any time before approval |
| Record the cancellation (kept as a status, never deleted) | 🟡 In progress | BR-PA-12, BR-PA-13 | Status change, no hard-delete |

## 10. Business-time & calendar

| Capability | Status | Spec ref | Notes |
|------------|--------|----------|-------|
| Measure all timing in business hours as the base unit | 🟡 In progress | BR-PA-16 | Modelling note only this slice: timestamps stored UTC, no timing enforced yet; days/half-days derived from working hours |
| Apply a shared working calendar (working hours per day, non-working weekdays) | ⬜ Not started | | |
| Pause timing on public holidays | ⬜ Not started | | |

## 11. Notifications

| Capability | Status | Spec ref | Notes |
|------------|--------|----------|-------|
| Send mobile push as the primary notification | ⬜ Not started | | |
| Notify on assignment, reminder, and escalation | ⬜ Not started | | |
| Send email for approvals | ⬜ Not started | | Open question — to be confirmed with client |

## 12. Dashboard & reporting

| Capability | Status | Spec ref | Notes |
|------------|--------|----------|-------|
| Show per-team task counts | ⬜ Not started | | |
| Show KPI breaches | ⬜ Not started | | |
| Attribute delays and work to the right team | ⬜ Not started | | |
| Count each step only once | ⬜ Not started | | |
| Scope views to a user's role | ⬜ Not started | | |

## 13. Audit trail

| Capability | Status | Spec ref | Notes |
|------------|--------|----------|-------|
| Keep an immutable, append-only record of every action | 🟡 In progress | BR-PA-14, BR-PA-15 | Append-only audit per action; never edited/deleted |
| Show a full timeline for each task | ⬜ Not started | | |

## 14. Integrations

| Capability | Status | Spec ref | Notes |
|------------|--------|----------|-------|
| Trigger tasks from incoming webhooks | ⬜ Not started | | |
| Send outgoing webhooks to other systems (e.g. SAP) | ⬜ Not started | | |

## 15. Platform & hosting

| Capability | Status | Spec ref | Notes |
|------------|--------|----------|-------|
| Run self-hosted on the organisation's own infrastructure | ⬜ Not started | | |
| Stay domain-agnostic — handle any task type, not one department | ⬜ Not started | | |
| Change flows, teams, and thresholds by configuration, not code | ⬜ Not started | | |

---

*Prepared for client review (Arjun). Status reflects build progress; capability names are
deliberately plain. As the specification formalises, the Spec ref column will be filled in
and statuses updated.*

---

## Out of scope / not planned

To keep the system focused, it deliberately does **not** do the following. These are handled
by other systems that this engine integrates with, rather than duplicating:

- **Storing inventory, SKU, or vendor data** — this data stays in its own system and is
  reached via webhooks; it is not owned or copied here.
- **Being an inventory or warehouse system** — this engine routes approvals only.
- **Per-user / per-seat licensing** — the system is self-hosted with no per-seat fees.
- **Replacing SAP or other systems of record** — those remain authoritative; we integrate
  with them, we do not replace them.
