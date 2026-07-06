# Feature Specification: Plain Approval Flow (Slice 1)

**Feature Branch**: `001-plain-approval-flow`

**Created**: 2026-07-06

**Status**: Draft

**Input**: First demoable slice — the plain approval flow only. Raise a one-off task
from a template, route it through sequential team-owned steps, one assignee approves per
step, advancing until the final approval closes the task. Records task, assignment, and
audit. Escalation, KPI clock, reject/boss-review, request-for-information, delegation,
recurring triggers, webhooks, dashboard, and notifications (beyond the minimum) are
explicitly deferred to later slices.

---

## Scope of this slice

**In scope**

- Raise a task from a task type / template, on demand (one-off).
- Sequential steps, each owned by a team; the task advances to the next step only when the
  current step is approved.
- A single assignee per step; that assignee approves, or the task cannot advance.
- Approve → advance to the next step; approve at the final step → task approved and closed.
- Basic task, assignment, and audit records for these actions.
- The following decisions are baked in even at this slice: segregation of duties (D-3),
  details locked after submission / notes only (D-6), restart-from-zero on edit & resubmit
  (D-2), cancellation before approval with no hard-delete (D-4), and business-hours as the
  base timing unit as a **modelling note only** (D-1 — no timing is enforced in this slice).

**Explicitly out of scope (deferred to later slices)**

- Silence-track escalation and PIC #1 → #2 → boss fallback.
- KPI clock / management escalation ladder.
- Reject and boss-review flow.
- Request-for-information (pause clocks, creator SLA, escalate to creator's boss, withdraw).
- Delegation / forwarding (ad-hoc and standing).
- Recurring / scheduled triggers (this slice is on-demand only).
- Webhooks (incoming and outgoing), dashboard/reporting, and any notification beyond the
  minimum "the current assignee is told the task is now theirs".
- Restricted-creator enforcement (D-5) — deferred; see Assumptions.
- SLA/KPI timing, deadlines, reminders, and business-hours computation logic (D-1 enforcement).

---

## User Scenarios & Testing *(mandatory)*

### User Story 1 - Raise and approve a task through to closure (Priority: P1)

A designated requester raises a task from a task type's template. The task lands with the
first step's assignee. That assignee approves, and the task advances to the next step's
assignee. When the assignee of the final step approves, the task is marked approved and
closed. Every action is recorded.

**Why this priority**: This is the whole point of the system — routing an approval from
raise to closure. Without it there is no product. It is the minimum that demonstrates a
working engine to the client.

**Independent Test**: Configure a task type with two sequential steps (Team A then Team B),
each with one assignee. Raise a task; confirm it sits with Team A's assignee. Approve as
Team A's assignee; confirm it moves to Team B's assignee. Approve as Team B's assignee;
confirm the task is approved and closed, and that an audit trail records each transition.

**Acceptance Scenarios**:

1. **Given** a task type with an ordered list of steps, **When** a requester raises a task
   from it, **Then** the task is created at step 1 and assigned to step 1's assignee, and a
   "task created" and an "assigned" audit record are written.
2. **Given** a task sitting at a non-final step, **When** that step's assignee approves,
   **Then** the task advances to the next step, a new assignment is created for the next
   step's assignee, and "approved" and "assigned" audit records are written.
3. **Given** a task sitting at the final step, **When** that step's assignee approves,
   **Then** the task is marked approved and closed, no new assignment is created, and
   "approved" and "closed" audit records are written.
4. **Given** a task sitting at a step, **When** a user who is not the current assignee
   attempts to approve, **Then** the action is rejected and the task does not advance.

### User Story 2 - Segregation of duties: the requester can never approve their own task (Priority: P1)

The person who raised a task is never allowed to act as an approver of that task at any
step, enforced by the server regardless of how steps are configured.

**Why this priority**: A non-negotiable control (Constitution Principle X / D-3). Approval
must always come from someone other than the requester. Shipping the approval flow without
this would let a requester self-approve, defeating the system's purpose.

**Independent Test**: Raise a task whose current step's assignee resolves to the requester.
Attempt to approve as the requester; confirm the server refuses. Confirm a task type cannot
route a task to its own requester as the sole approver of a step.

**Acceptance Scenarios**:

1. **Given** a task raised by user R, **When** R attempts to approve any step of that task,
   **Then** the server rejects the action with a segregation-of-duties error and no approval
   is recorded.
2. **Given** a step whose only configured assignee is the requester, **When** the task
   reaches that step, **Then** the task cannot be advanced by the requester and the
   condition is surfaced as a configuration/routing error rather than silently allowing
   self-approval.

### User Story 3 - Cancel a task before it is approved (Priority: P2)

An authorised user cancels a task at any time before it is approved. The task is marked
cancelled and stops routing; nothing is deleted.

**Why this priority**: Client-requested (D-4) and simple. It prevents dead tasks lingering
in someone's queue and demonstrates the "no hard-delete, status-change only" data rule.

**Independent Test**: Raise a task, then cancel it while it sits at a step. Confirm its
status becomes cancelled, it no longer appears as an active assignment, the record is
retained, and a cancellation audit record exists. Confirm an already-closed task cannot be
cancelled.

**Acceptance Scenarios**:

1. **Given** an open task at any step before approval, **When** an authorised user cancels
   it, **Then** the task status becomes "cancelled", its active assignment is closed, and a
   "cancelled" audit record is written.
2. **Given** a task that is already approved and closed, **When** a user attempts to cancel
   it, **Then** the action is refused.
3. **Given** a cancelled task, **When** the record is later read, **Then** it still exists in
   full (no hard-delete) with its cancellation recorded.

### User Story 4 - Locked details; edit & resubmit restarts from step 1 (Priority: P2)

After submission a task's request details are locked — users may add notes but not alter the
submitted details in place. If the requester genuinely changes the request and resubmits,
the task restarts from step 1 and all prior approvals are discarded.

**Why this priority**: Enforces D-6 (locked details) and D-2 (restart-from-zero), which
protect the integrity of an approval: an approval only ever applies to the exact version
that was approved. Baked in now so later slices inherit correct behaviour.

**Independent Test**: Raise a task and approve step 1. Attempt to edit the submitted details
in place; confirm it is blocked (notes are allowed). Perform an explicit edit-and-resubmit
that changes the request; confirm the task returns to step 1, prior approvals are discarded,
and the restart is recorded.

**Acceptance Scenarios**:

1. **Given** a submitted task, **When** a user tries to change the submitted request details
   in place, **Then** the change is refused; adding a note is permitted.
2. **Given** an open task with one or more prior approvals, **When** the requester edits the
   request details and resubmits, **Then** the task restarts at step 1, all prior approvals
   are discarded, and an "edited & resubmitted / restarted" audit record is written.
3. **Given** an action that adds only a note and does not change request details, **When** it
   is applied, **Then** the task does NOT restart and stays at its current step.

### Edge Cases

- **Single-step flow**: A task type with exactly one step is valid; approving that one step
  approves and closes the task (there is no "next step").
- **Approval by a stale actor**: A user who was the assignee before an edit-and-resubmit
  restart attempts to approve the old step — the action must be evaluated against the task's
  current step/assignment, not the superseded one.
- **Double approval**: The same assignee approves the same step twice (e.g. duplicate
  submit) — only the first is effective; the second must not double-advance the task.
- **Cancel race**: A cancel and an approval arrive for the same task at the same step — the
  task resolves to exactly one terminal outcome (cancelled or advanced/closed), never both.
- **Requester equals a downstream assignee**: The requester is validly an approver on some
  other unrelated task but is the requester here — segregation of duties applies only to
  their own task (US2), not to tasks they did not raise.

---

## Requirements *(mandatory)*

### Functional Requirements

- **FR-001**: The system MUST allow a designated requester to raise a task from a task type
  on demand (one-off), capturing that task type's template field values against the task.
- **FR-002**: The system MUST record, for every task, its task type, its requester, its
  captured template values, and its creation time.
- **FR-003**: A task MUST follow the ordered sequence of steps defined by its task type,
  beginning at the first step.
- **FR-004**: Each step MUST be owned by exactly one team.
- **FR-005**: For this slice, each step MUST have exactly one assignee who must act for the
  task to advance. (The fuller PIC #1 → #2 → boss fallback list is deferred.)
- **FR-006**: When a task enters a step, the system MUST create a single active assignment
  for that step's assignee.
- **FR-007**: A task MUST advance to step *n+1* only after step *n* is approved. Parallel or
  concurrent approvals are out of scope.
- **FR-008**: On approval at a non-final step, the system MUST advance the task to the next
  step and create the next assignment.
- **FR-009**: On approval at the final step, the system MUST mark the task approved and
  closed and create no further assignment.
- **FR-010**: A task MUST have exactly one active assignment at any time.
- **FR-011**: Only the current step's active assignee MUST be able to approve the current
  step; the system MUST reject an approval from anyone else.
- **FR-012**: The system MUST prevent the task's requester from acting as an approver of
  their own task at any step, enforced server-side (segregation of duties, D-3).
- **FR-013**: The system MUST NOT allow a task to be advanced by an approval from its
  requester even if configuration would route the step to them; such a condition MUST be
  surfaced as an error rather than permitting self-approval.
- **FR-014**: After submission, the system MUST lock a task's request details against
  in-place changes; users MAY add notes/comments but MUST NOT alter the submitted request in
  place (D-6).
- **FR-015**: When the requester edits the request details and resubmits, the system MUST
  restart the task at step 1 and discard all prior approvals (D-2). An action that adds only
  a note (no change to request details) MUST NOT trigger a restart.
- **FR-016**: The system MUST allow an authorised user to cancel a task at any time before it
  is approved; cancellation MUST set the task status to cancelled and close its active
  assignment (D-4).
- **FR-017**: The system MUST refuse cancellation of a task that is already approved/closed.
- **FR-018**: The system MUST NOT hard-delete any task, assignment, or audit record;
  cancellation and all other outcomes are status changes with the record retained.
- **FR-019**: The system MUST write exactly one append-only audit record for every
  state-changing action in this slice: task created, assignment created/assigned, approved,
  advanced (step from/to), approved-and-closed, cancelled, and edited-&-resubmitted/restart.
- **FR-020**: Each audit record MUST capture the event type, the actor (a user, or "system"
  for automated transitions), a timestamp, and relevant detail (e.g. from-step and to-step).
- **FR-021**: Audit records MUST be append-only — never updated or deleted.
- **FR-022**: The system MUST notify the current assignee when a task becomes theirs (the
  minimum notification for the flow to function). Richer notification behaviour is deferred.
- **FR-023**: The system MUST store all timestamps in UTC. No SLA/KPI deadline, reminder, or
  escalation behaviour exists in this slice; business-hours timing is a modelling note for
  later slices only (D-1).

### Key Entities *(include if feature involves data)*

- **Task type**: A reusable flow definition — its template fields and its ordered list of
  steps. Configuration, not code.
- **Step**: One stage in a task type's flow, owned by exactly one team, with a single
  assignee for this slice. Has a sequence position.
- **Team**: The owning group of a step. (Team hierarchy/boss is modelled minimally here;
  boss-of-boss and escalation targets are deferred.)
- **User**: A person who can raise tasks (requester) and/or be a step assignee (approver).
- **Task**: A running instance of a task type — its requester, captured template values,
  current step, and status (open / approved / cancelled / closed).
- **Assignment**: The single live "ball in someone's court" for the current step — who is
  responsible and the outcome (pending / approved / superseded / cancelled).
- **Audit record**: An append-only entry for every state change — event type, actor,
  timestamp, and detail (including step moved from/to). The system of record for the task's
  timeline.

---

## Business Rules (testable) *(this slice)*

> These are **slice-scoped** rule IDs (prefix `BR-PA-*`, "Plain Approval"), deliberately
> distinct from the source specification's global `BR-1..BR-18` so they can be mapped onto
> the feature checklist's Spec ref column without collision. Traceability to source is noted
> in brackets.

- **BR-PA-1**: A task can be raised from a task type on demand (one-off); the task records
  its task type, requester, captured template values, and creation time. [FR-3, D-5-context]
- **BR-PA-2**: A task begins at step 1 and proceeds through the task type's steps in defined
  order; steps are configuration and require no code change to add/remove/reorder. [FR-5–8,
  Constitution VI]
- **BR-PA-3**: Each step is owned by exactly one team and (this slice) has exactly one
  assignee who must act for the task to advance. [FR-6, FR-11 simplified]
- **BR-PA-4**: A task advances to the next step only after the current step is approved.
  [source BR-1, FR-7, FR-18]
- **BR-PA-5**: Approval at a non-final step advances the task to the next step and assigns it
  to that step's assignee. [FR-18]
- **BR-PA-6**: Approval at the final step marks the task approved and closed; no further
  assignment is created. [FR-19]
- **BR-PA-7**: Only the current step's active assignee may approve that step; an approval
  from anyone else is rejected and the task does not advance. [FR-17 subset]
- **BR-PA-8**: A task has exactly one active assignment at any time. [source BR-15, FR-16j
  subset]
- **BR-PA-9**: The requester of a task can never act as an approver of that task at any step;
  the server rejects any such approval, and a configuration that would make the requester the
  sole approver of a step is treated as an error, not a self-approval. [D-3, Constitution X]
- **BR-PA-10**: After submission, a task's request details are locked; users may add notes
  but may not alter the submitted request in place. [D-6]
- **BR-PA-11**: If the requester edits the request details and resubmits, the task restarts
  at step 1 and all prior approvals are discarded; a note-only addition that does not change
  request details does not restart the flow. [D-2]
- **BR-PA-12**: A task may be cancelled by an authorised user at any time before it is
  approved; cancellation sets status to cancelled and closes the active assignment. An
  already-approved/closed task cannot be cancelled. [D-4]
- **BR-PA-13**: No task, assignment, or audit record is ever hard-deleted; cancellation and
  every other outcome is a retained status change. [D-4, Constitution — Code Quality & Data]
- **BR-PA-14**: Every state-changing action in this slice (created, assigned, approved,
  advanced, approved-and-closed, cancelled, edited-&-resubmitted/restart) writes exactly one
  append-only audit record capturing event type, actor (or "system"), UTC timestamp, and
  relevant detail (including step from/to). [source BR-9, FR-34–37]
- **BR-PA-15**: Audit records are append-only — never updated or deleted. [FR-36]
- **BR-PA-16**: Timestamps are stored in UTC; this slice enforces no deadlines, reminders, or
  escalation. Business-hours-as-base-unit is recorded as a modelling note for later slices,
  not implemented here. [D-1, FR-16a — deferred enforcement]

---

## Success Criteria *(mandatory)*

### Measurable Outcomes

- **SC-001**: A requester can raise a task and see it land with the first approver in a
  single action, 100% of the time, for any configured task type of 1..N steps.
- **SC-002**: A task with N sequential steps reaches "approved and closed" after exactly N
  approvals by the correct assignees, and not before — verifiable for N = 1, 2, and 3.
- **SC-003**: 100% of approval attempts by a non-assignee, and 100% of approval attempts by
  the task's own requester, are refused with no state change.
- **SC-004**: 100% of state-changing actions produce exactly one audit record; the full
  timeline of a task can be reconstructed from audit records alone.
- **SC-005**: A task can be cancelled before approval and can never be cancelled after
  closure; cancelled tasks remain fully retrievable (no data lost to deletion).
- **SC-006**: Editing and resubmitting a task's details returns it to step 1 with zero prior
  approvals retained, 100% of the time; note-only additions never cause a restart.

---

## Assumptions

- **Configuration exists**: At least one task type with an ordered list of steps, each step
  owning a team and naming a single assignee, is configured before a task is raised. Building
  the configuration UI/admin is not part of this slice; seed/fixture configuration is
  sufficient to demonstrate the flow.
- **Restricted creators (D-5) deferred**: This slice does not enforce *who* may raise a task
  (D-5 was intentionally excluded from the bake-in list). Any authenticated user acting as a
  requester is accepted for now; creator restriction is a later slice. Segregation of duties
  (D-3) is still enforced regardless.
- **Single assignee per step**: A deliberate simplification of the source model's PIC
  fallback list (PIC #1 → #2 → boss). The data model may still represent it as an ordered
  list of length one; fallback behaviour is deferred.
- **Minimal notification only**: "Assignee is notified the task is theirs" is assumed to be
  satisfiable by the simplest available channel; the mobile-first push design and
  reminder/escalation notifications are deferred.
- **No timing enforcement**: Per D-1, business hours is the intended base unit, but because
  no SLA/KPI/escalation exists in this slice, no working-calendar computation is built here.
  Timestamps are UTC so later slices can layer business-time on top without rework.
- **Boss-review / resume-in-place not reachable**: Because reject and request-for-information
  are out of scope, the only path that returns a task toward step 1 in this slice is the
  requester's own edit-and-resubmit (D-2). FR-21 resume-in-place (boss override) is a later
  slice and does not apply here.

---

## Dependencies

- The approval engine platform (the trusted process engine) provides step execution and
  human-task waiting; this slice builds the custom raise/approve/cancel/audit behaviour on
  top. (Per the constitution, engine capabilities are reused rather than rebuilt; naming the
  specific platform is an implementation concern handled in `/speckit-plan`.)
- Authoritative decisions: `docs/source/decisions.md` (D-1, D-2, D-3, D-4, D-6). Where the
  older source documents conflict, `decisions.md` governs.

---

## Conflicts flagged (source docs vs decisions.md)

Recorded for traceability; all are resolved by `decisions.md` (authoritative) and none block
this slice:

- **C-1 (business-time scope)**: Q&A says business-time is "only for the KPI's"; D-1 applies
  it to both the personal and KPI clocks by default (per-clock configurable). Resolution: D-1
  wins. Not exercised in this slice (no timing enforced).
- **C-2 (restart vs resume)**: Source FR-21 resumes in place after a boss override; D-2
  restarts from zero on requester edit/resubmit. These are different triggers and D-2
  reconciles them. Only D-2's edit/resubmit rule is in scope here; boss override is deferred.
- **C-3 (cancellation)**: The source specification has no explicit cancel action; Q&A and D-4
  add it. Additive, not a contradiction; included in this slice.
