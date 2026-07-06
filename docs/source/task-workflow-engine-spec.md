**Task Workflow Engine — Process & Requirements Specification**

**Status:** Draft for development

**Audience:** Engineering team

**Purpose:** Define the behaviour, data model, and business rules for a configurable task routing and approval engine (e.g. installer invoice approval, staff claims).

## **1\. Overview**

The system runs **tasks** through a **configurable chain of approval steps**. Each task type defines its own flow: an ordered list of steps, where each step is owned by a team and routed to that team's person-in-charge (PIC). If a PIC does not respond within their time window, the task automatically escalates to the next person, and ultimately to the team boss. Tasks can be triggered manually (one-off) or on a recurring schedule.

The engine is built around three ideas:

1. **Flows are data, not code.** A task type is an ordered list of steps of any length — one team or many. Adding, removing, or reordering teams is a configuration change, never a code change.

2. **Escalation is silence-driven.** Each assignment carries its own response deadline. If the assignee goes silent, the task hops to the next responsible person with a fresh deadline.

3. **Every state transition is a record.** Each hop, decision, and escalation creates an assignment row, giving a complete audit trail and the raw data for dashboards.

## **2\. Glossary**

| Term | Definition |
| :---- | :---- |
| **Task type** | A reusable definition of a flow (e.g. "Tech invoice approval"). Holds the template and the ordered steps. |
| **Step** | One stage in a flow, owned by a single team. A flow has 1..N steps. |
| **Task** | A running instance of a task type, created by a trigger. |
| **PIC (Person In Charge)** | The person assigned to act on a step. Each step has an ordered list: PIC \#1, PIC \#2, … then the team boss. |
| **Assignment** | A single live "ball in someone's court" with its own response deadline. One task generates many assignments over its life. |
| **Response SLA** | Per-assignment time window to respond (e.g. 24h). Drives escalation. |
| **Step KPI** | Hard cap on total time for a step. Breaching it escalates the task up the management chain; also reported on the dashboard. |
| **Requester** | The person who raised the task; the final fallback if a task is rejected up the chain. |

## **3\. Functional Requirements**

### **3.1 Task behaviour**

* **FR-1** A task type MUST be markable as **recurring** or **non-recurring**.

* **FR-2** A recurring task type MUST support a schedule (calendar-style). Every trigger MUST create a new, independent task instance.

* **FR-3** A non-recurring task type MUST be creatable manually on demand.

* **FR-4** If a recurring trigger fires while a previous instance of the same task type is still open, the new instance MUST still be created. Instances **stack** — each runs concurrently and independently, with its own steps, assignments, and SLA clocks. The engine MUST NOT skip, merge, or block a scheduled instance because an earlier one is unresolved.

### **3.2 Task types & configurable flows**

* **FR-5** Each task type MUST define an ordered list of steps (sequence), of any length (1, 2, 3, … N teams).

* **FR-6** Each step MUST be associated with exactly one team.

* **FR-7** Steps MUST execute sequentially: a task advances to step *n+1* only once step *n* is approved. Parallel/concurrent approvals are **out of scope** (confirmed).

* **FR-8** Steps MUST be reorderable and add/removable via configuration, without code changes.

### **3.3 Templates**

* **FR-9** Each task type MUST have a standard template defining the fields captured for every task of that type.

* **FR-10** Template field values MUST be stored per task instance.

### **3.4 Routing, PIC fallback & escalation**

* **FR-11** Each step MUST define an ordered list of assignees: PIC \#1, PIC \#2, … and finally the team boss (flagged accordingly).

* **FR-12** When a task enters a step, it MUST be assigned to PIC \#1 with a response deadline (now \+ response\_sla\_hours, default 24h).

* **FR-13** If the current assignee does not respond before the deadline, the task MUST automatically escalate to the next assignee in the list, with a **fresh** deadline.

* **FR-14** Escalation order within a step is: **PIC \#1 → PIC \#2 → team boss.**

* **FR-15** The response SLA is **per assignment** (per person), not per step. Each hop resets the clock. This is the **silence track** (handles an individual not responding).

* **FR-16** The step KPI (total target time for the whole step) is a **hard cap**, not just a reporting metric. When a step's total elapsed business time exceeds step\_kpi\_hours, the task MUST escalate **up the management hierarchy**: to the **team boss**, then to the **boss of boss** (parent team's boss), and finally — if still unresolved past KPI — to a single **company-wide top escalation authority** (see FR-25a). This is the **KPI track** (handles the whole step taking too long), and it runs independently of and in parallel with the silence track in FR-13/14.

* **FR-16a** SLA and KPI windows MUST be measured in **elapsed business hours**, not wall-clock time. The clock advances only during configured working hours and pauses on weekends, public holidays, and outside working hours. A **business day** (used by the half-day intervals) equals the configured working hours in one day, so **half a business day \= half that** (e.g. 4 business hours on an 8-hour working day). The working calendar — working hours per day, non-working weekdays, and the holiday list — is deployment configuration referenced from org\_setting.

* **FR-16b** The step KPI clock starts when the task **enters** the step and runs until the step is approved. It counts total business time across everything inside the step — silence-track hops, rejection, boss review, resume-in-place, and forwarding — and is **never reset** by any of them. The one thing that can **pause** it (not reset it) is a **pending info-request to the creator** (FR-23g): the clock stops while the ball is in the creator's court and resumes when they reply. The KPI track terminates at the company top escalation authority (FR-25a); it does not climb indefinitely.

#### ***Delegation & forwarding***

* **FR-16c** An assignee MUST be able to **forward (delegate)** their current assignment to another user. The delegate acts in the original assignee's place.

* **FR-16d** A forward MUST carry the **step KPI unchanged** — it does not reset or pause the step KPI clock; only the responsible person changes. The delegate's own response-SLA window is set per FR-16i.

* **FR-16e** Once the delegate acts (approve / reject / request info), the task **continues in the normal flow** exactly as if the original assignee had acted.

* **FR-16f** Forwarding MUST be available two ways: **ad hoc** (a single task/assignment) and **standing** (a delegation rule that auto-routes assignments for a date range — e.g. while on leave), scoped to **all tasks** or a chosen task type.

* **FR-16g** While a standing delegation is active, any assignment that would be routed to the delegator MUST instead be routed to the delegate on the delegator's behalf. When the delegation ends, routing reverts automatically.

* **FR-16h** Delegated actions MUST be recorded in the audit trail showing both the **acting user** and the **person they acted for**.

* **FR-16i** A **delegated** assignment uses a shorter response SLA of **half a business day**. If the delegate is silent past it, the silence track escalates along the **original step's assignee list** (the original flow): the step's next PIC, then the team boss — not the delegate's own hierarchy. The delegate merely stands in for the current holder; escalation always follows the step's defined chain. (The step KPI is unaffected by delegation; only this per-person silence window is shortened.)

#### ***Reconciling the two tracks***

* **FR-16j** A task MUST have **exactly one active assignment** (one holder) at any time. Escalation supersedes the current assignment and creates one new one — never two live approvers.

* **FR-16k** Escalation positions form a **single ordered authority ladder**: PIC \#1 (1) → PIC \#2… (2) → team boss (3) → boss of boss (4) → company top authority (5). Both tracks advance the task along this one ladder.

* **FR-16l** A task only ever moves **up** the ladder. Each track trigger proposes a target rung; the engine moves the task to max(current\_rung, proposed\_rung). A trigger whose target rung is **equal to or lower than** the current rung MUST be **deduped** — recorded in the audit trail / dashboard (e.g. the KPI breach still counts) but MUST NOT create a new assignment or send a duplicate notification.

* **FR-16m** Track ranges on the ladder: the **silence track** advances rungs 1 → 3 only (caps at team boss; one rung per unanswered response\_sla\_hours). The **KPI track** targets rung 3 on first hard-cap breach, then one further rung per **\`kpi\_escalation\_interval\` of half a business day** of continued overdue time (→ 4, → 5). Above team boss, only the KPI track moves the task (there is no further PIC list).

* **FR-16n** On any upward move: the previous assignment is set to superseded, one new assignment is created at the target rung with a **fresh response SLA**, escalation\_reason set to whichever track triggered (silence\_sla / step\_kpi\_breach / both if simultaneous), and the **step KPI clock is unchanged**. Any pending lower escalation (e.g. a queued silence hop) MUST be cancelled.

### **3.5 Approval & rejection**

* **FR-17** An assignee MUST be able to **approve** or **reject** an assignment.

* **FR-18** On approval at a non-final step, the task MUST advance to the next step.

* **FR-19** On approval at the final step, the task MUST be marked **approved** and closed.

* **FR-20** On rejection, the task MUST escalate to the **boss** for review (second-level decision).

* **FR-21** If the boss **upholds** the approval (overrides the rejection), the task MUST **resume at the same step it was rejected in** — not restart from the beginning.

* **FR-22** If the boss **also rejects**, the task MUST be returned to the **requester**.

* **FR-23** On resume-in-place after a boss override, the step **KPI clock is NOT reset** — it continues from before the rejection detour (per FR-16b), so the detour cannot buy extra step time. The new assignment created on resume gets its own **fresh response SLA** (consistent with the per-assignment rule in FR-15); only the per-person silence clock is fresh, never the step KPI.

* **FR-23a** An assignee MUST be able to add **comments** and **file attachments** to their action on any step. On **approval**, both are **optional**.

* **FR-23b** On **rejection**, a comment is **mandatory** — the system MUST block submission of a rejection without a reason. Attachments remain optional on rejection.

* **FR-23c** Comments and attachments MUST be retained against the specific assignment/action that produced them and MUST appear in the task's audit trail and timeline.

* **FR-23d** Attachment handling: accepted file types are **PDF, images (JPG/PNG/etc.), and Office documents (Word, Excel, PowerPoint)**, **max size 50 MB per file**. In addition, an assignee/creator MAY attach a **link (URL)** instead of a file. Attachments MUST be access-controlled to the same users who can view the task.

#### ***Request for information***

* **FR-23e** Besides approve and reject, an assignee MUST be able to **request more information from the creator (requester)** — a clarification query that is **not** a rejection.

* **FR-23f** A request-for-information MUST keep the task at the **same step with the same assignee**; the task is not routed elsewhere. The assignee resumes their approve/reject decision once the creator replies.

* **FR-23g** While an info-request is **pending** (with the creator or, after escalation, the creator's boss), the **step KPI clock and the assignee's response SLA both PAUSE** — the wait is on the creator side, so the step does not accrue time against either clock. Both resume only when the **info is provided**. *(This revises the earlier assumption that an info-request never affects the KPI: it does not reset the KPI, but it does pause it.)*

* **FR-23h** The creator MUST be notified of the query (mobile push) and MUST be able to reply with a comment and optional attachments. Both the query and the reply MUST be recorded in the audit trail.

* **FR-23i** When an info-request is raised, the creator MUST be given their **own response deadline** (a creator-side SLA, in business time) by which they must reply. Reminders and breach notifications apply to this deadline as they do for any assignment.

* **FR-23j** When the info is provided (by the creator or the creator's boss), the paused clocks resume from where they stopped (the KPI is never reset; pause-and-resume only).

* **FR-23k** If the **creator misses** their response deadline, the info-request MUST escalate to the **creator's boss** (the boss of the creator's team), who is notified and given their own response deadline. If the **creator's boss also takes no action**, the info-request **stays with the creator's boss** — there is no further escalation up the creator's chain.

* **FR-23l** The approver who raised an info-request MAY **withdraw** it at any time and proceed to decide (approve / reject) with the information available. On withdrawal, the info-request is closed, the paused **step KPI and assignee SLA resume** from where they stopped, and the task is no longer parked. This is the escape valve that prevents a task parking indefinitely on a silent creator side.

### **3.6 Team structure**

* **FR-24** The system MUST model a team hierarchy: team members, a team boss, and a boss-of-boss (parent team).

* **FR-25** Each team MUST have a designated boss who serves as the final escalation target for that team's steps and the reviewer for rejections.

* **FR-25a** The system MUST support a single, company-wide **top escalation authority** — one designated person — as the final target of the KPI escalation track when team boss and boss-of-boss have not resolved the task. This is a single org-level configuration, not per team.

### **3.7 Notifications & reminders**

* **FR-26** The system MUST notify the current assignee when a task lands in their court.

* **FR-27** The system MUST support configurable reminders before a response deadline.

* **FR-28** The system MUST send notifications on escalation and SLA breach.

* **FR-29** Notification delivery MUST prioritise **mobile app push**.

### **3.8 Dashboard**

* **FR-30** The dashboard MUST show task counts per team.

* **FR-31** The dashboard MUST show KPI performance (did steps land inside their target time).

* **FR-31a** A **KPI breach is attributed to the team that owns the step** and counted as a **breach against that team** in the dashboard. Blame is at the team level, not the individual — the team is collectively accountable for clearing its step within KPI. (The individual who held the task at breach time is still traceable via the audit trail, but is not the unit the breach is charged to.)

* **FR-31b** A step is counted as breached **at most once**, fixed at the **first breach within that step** (the first person to exceed their KPI/SLA). When that first breach then escalates the task to the next person or to the boss, those onward escalations MUST **NOT** be counted as additional breaches — the first miss is the step's single breach, and it carries the breach timestamp and team attribution. The dashboard de-duplicates per step so escalation never inflates the breach count.

* **FR-32** The dashboard SHOULD surface breached / overdue tasks, average time per step (to identify bottlenecks), and current escalation levels.

* **FR-33** Dashboard views SHOULD be role-scoped: a boss sees their team; a boss-of-boss sees subordinate teams.

### **3.9 Audit trail**

* **FR-34** The system MUST keep a complete, immutable audit trail for every task, recording every state change from creation to closure.

* **FR-35** Each audit entry MUST capture: the event type, the actor (the user who acted, or "system" for automated events such as escalations and reminders), a timestamp, and any relevant detail (decision, comment, step moved from/to, SLA snapshot).

* **FR-36** Audit entries MUST be **append-only** — never edited or deleted — so the trail is tamper-evident.

* **FR-37** The audit trail MUST cover at minimum: task created, assignment created/assigned, reminder sent, escalated (with level), approved, rejected, sent to boss review, resumed-in-place, returned to requester, and task closed.

* **FR-38** Each task's full audit trail MUST be viewable as a chronological timeline, and the data MUST feed the dashboard's breach history and time-per-step metrics.

## **4\. Process Flows**

### **4.1 Task lifecycle (end to end)**

Trigger fires (schedule or manual)  
        ↓  
Task created from template  
        ↓  
Step 1  (Team A)  → approve → advance  
        ↓  
Step 2  (Team B)  → approve → advance  
        ↓  
Step N  (final)   → approve  
        ↓  
Approved / closed

The number of steps is defined by the task type and may be 1 or many.

### **4.2 Inside a single step (two escalation tracks)**

A step runs **two clocks at once**. The *silence track* watches whether the current person responds; the *KPI track* watches the total time the whole step is taking.

**Silence track** — per-person response SLA (24h business time), resets each hop:

Task enters step  
        ↓  
Assigned to PIC \#1  (deadline \= now \+ 24h)  
        │  
        ├── responds in time → Approved → next step  
        │  
        └── deadline passes (silence) → escalate  
                    ↓  
            Assigned to PIC \#2  (fresh 24h)  
                    ↓  (if silent again)  
            Assigned to team boss  (fresh 24h)

**KPI track** — step total-time hard cap, runs in parallel from the moment the task enters the step:

Step KPI clock starts (task enters step)  
        ↓  
Total step time exceeds step\_kpi\_hours  → HARD CAP breached  
        ↓  
Escalate up the management hierarchy  
        ↓  
    Team boss  
        ↓  (still unresolved past KPI)  
    Boss of boss (parent team)

Either track can fire first. The silence track moves the task *across* the step's assignee list; the KPI track moves it *up* the org hierarchy.

**When both fire — one ladder, max-wins.** The two tracks share rung 3 (team boss) and are reconciled onto a single authority ladder: PIC \#1 → PIC \#2 → team boss → boss of boss → top authority. The task always sits at the highest rung reached and never moves down. A trigger whose target is equal to or below the current rung is logged (the breach still counts) but does not reassign or re-notify. Example: if silence has already escalated to the team boss and the KPI then breaches, the KPI's first target (team boss) equals the current rung → no move, audit only; only continued overdue time bumps it to boss of boss.

### **4.3 Reject path**

Approver rejects  
        ↓  
Boss reviews (second-level decision)  
        │  
        ├── boss approves → task resumes at the SAME step → continues down the chain  
        │  
        └── boss rejects again → returned to requester (revise & resubmit)

## **5\. Data Model**

Core entities. Field lists are indicative, not exhaustive.

### **task\_type**

Defines a reusable flow.

| Field | Type | Notes |
| :---- | :---- | :---- |
| id | uuid (PK) |  |
| name | string | e.g. "Tech invoice approval" |
| is\_recurring | bool |  |
| schedule | string | Calendar/cron expression; null if non-recurring |
| template\_fields | json | Field definitions for the task form |

### **step\_def**

One ordered step within a task type.

| Field | Type | Notes |
| :---- | :---- | :---- |
| id | uuid (PK) |  |
| task\_type\_id | uuid (FK → task\_type) |  |
| sequence | int | Order within the flow |
| team\_id | uuid (FK → team) | Owning team |
| response\_sla\_hours | int | Per-assignment deadline (default 24\) |
| step\_kpi\_hours | int | Hard-cap target for the step (business hours); drives KPI-track escalation and dashboard reporting |

### **step\_assignee**

Ordered PIC fallback list for a step.

| Field | Type | Notes |
| :---- | :---- | :---- |
| id | uuid (PK) |  |
| step\_def\_id | uuid (FK → step\_def) |  |
| user\_id | uuid (FK → app\_user) |  |
| order | int | 1 \= PIC \#1, 2 \= PIC \#2, … |
| is\_boss | bool | Final escalation target |

### **team**

Team hierarchy (boss-of-boss via parent\_team\_id).

| Field | Type | Notes |
| :---- | :---- | :---- |
| id | uuid (PK) |  |
| name | string |  |
| parent\_team\_id | uuid (FK → team) | Boss-of-boss chain; null at top |
| boss\_user\_id | uuid (FK → app\_user) | Team boss |

### **app\_user**

| Field | Type | Notes |
| :---- | :---- | :---- |
| id | uuid (PK) |  |
| name | string |  |
| team\_id | uuid (FK → team) |  |
| role | string | member / boss / admin |

### **task**

A running instance.

| Field | Type | Notes |
| :---- | :---- | :---- |
| id | uuid (PK) |  |
| task\_type\_id | uuid (FK → task\_type) |  |
| requester\_id | uuid (FK → app\_user) |  |
| current\_step\_id | uuid (FK → step\_def) | Where the task currently sits |
| current\_step\_started\_at | timestamp | When the task entered the current step; start of the step KPI clock |
| step\_kpi\_due\_at | timestamp | Step KPI hard-cap deadline (business time); breaching it drives the vertical KPI-track escalation. Pushed forward by any paused interval (FR-23g) so paused time never counts against the cap |
| kpi\_clock\_state | string | running / paused — paused while a creator info-request is pending |
| status | string | open / approved / returned / closed |
| template\_data | json | Captured field values |
| created\_at | timestamp |  |

### **task\_assignment**

A single live "ball in court" — the heart of routing and escalation. One row per hop.

| Field | Type | Notes |
| :---- | :---- | :---- |
| id | uuid (PK) |  |
| task\_id | uuid (FK → task) |  |
| step\_def\_id | uuid (FK → step\_def) | Which step this assignment serves |
| assignee\_id | uuid (FK → app\_user) | The user currently responsible (may be a delegate) |
| acting\_for\_user\_id | uuid (FK → app\_user) | Set when assignee\_id is acting as a delegate; null otherwise |
| escalation\_level | int | Position reached: 1 \= PIC \#1, 2 \= PIC \#2, 3 \= team boss, 4 \= boss of boss, 5 \= company top authority |
| escalation\_reason | string | Why this assignment was created: initial / silence\_sla / step\_kpi\_breach / rejection\_review / forward |
| due\_at | timestamp | Response deadline (assigned\_at \+ SLA); preserved unchanged on a forward (FR-16d) |
| status | string | pending / approved / rejected / timed\_out / superseded |
| decision | string | approve / reject |
| comment | text | Optional on approval; **mandatory on rejection** (FR-23b) |
| responded\_at | timestamp | Null until acted on |

### **Supporting tables**

* **\`notification\`** — one row per push/reminder against a task\_assignment; type ∈ {assigned, reminder, escalation, breach, info\_request, info\_reply, info\_overdue, forwarded}; channel (mobile push primary); sent\_at. Drives the mobile-first notification requirement.

* **\`org\_setting\`** (or equivalent config) — holds the single top\_escalation\_user\_id (FR-25a), the working-calendar reference (FR-16a), the default **creator info-request SLA** (FR-23i), the **\`kpi\_escalation\_interval\`** (default half a business day, FR-16m), and the **\`delegated\_response\_sla\`** (default half a business day for delegated assignments, FR-16i). All may be overridable per task type / step.

### **delegation**

Standing delegation rules — auto-route assignments during leave, etc. (FR-16f, FR-16g). Ad-hoc one-off forwards do not need a rule; they are applied directly to a single task\_assignment (with acting\_for\_user\_id).

| Field | Type | Notes |
| :---- | :---- | :---- |
| id | uuid (PK) |  |
| delegator\_id | uuid (FK → app\_user) | Person being covered |
| delegate\_id | uuid (FK → app\_user) | Person covering |
| scope | string | all\_tasks / task\_type |
| task\_type\_id | uuid (FK → task\_type) | Null when scope \= all\_tasks |
| start\_at | timestamp |  |
| end\_at | timestamp | Routing reverts after this |
| status | string | active / ended / cancelled |

### **attachment**

Files or links attached to a step action (FR-23a–d).

| Field | Type | Notes |
| :---- | :---- | :---- |
| id | uuid (PK) |  |
| task\_id | uuid (FK → task) | For task-level access scoping |
| task\_assignment\_id | uuid (FK → task\_assignment) | The action this attachment belongs to |
| uploaded\_by | uuid (FK → app\_user) |  |
| kind | string | file / link |
| file\_name | string | Original name (file only) |
| mime\_type | string | Validated against accepted types: PDF, image, Office doc (file only) |
| size\_bytes | int | Enforced against the 50 MB per-file limit (file only) (FR-23d) |
| storage\_url | string | Pointer to object storage (file only) |
| link\_url | string | The URL (link only) |
| created\_at | timestamp |  |

Note: attachment may also reference an info\_request (add nullable info\_request\_id) so the creator's reply can carry files or links.

### **info\_request**

A clarification query from an assignee to the creator (FR-23e–i).

| Field | Type | Notes |
| :---- | :---- | :---- |
| id | uuid (PK) |  |
| task\_id | uuid (FK → task) |  |
| task\_assignment\_id | uuid (FK → task\_assignment) | The assignment that raised the query; task stays here |
| requested\_by | uuid (FK → app\_user) | The assignee |
| requested\_from | uuid (FK → app\_user) | The creator/requester (initial responder) |
| current\_responder\_id | uuid (FK → app\_user) | Who must respond now: creator, then creator's boss |
| responder\_level | int | 1 \= creator, 2 \= creator's boss |
| question | text | The clarification asked |
| due\_at | timestamp | Current responder's deadline (business time); reset when escalated to the boss (FR-23i, FR-23k) |
| response | text | Reply; null until answered |
| status | string | pending / escalated\_to\_boss / parked / answered / withdrawn |
| answered\_at | timestamp | Null until answered |
| created\_at | timestamp |  |

### **audit\_event**

Append-only log of every transition — the system of record for each task's history (FR-34 to FR-38).

| Field | Type | Notes |
| :---- | :---- | :---- |
| id | uuid (PK) |  |
| task\_id | uuid (FK → task) | Always set |
| task\_assignment\_id | uuid (FK → task\_assignment) | Null for task-level events |
| event\_type | string | created / assigned / reminder\_sent / escalated\_silence / step\_kpi\_breach / escalated\_kpi / escalated\_top\_authority / info\_requested / info\_escalated / info\_parked / info\_provided / info\_withdrawn / forwarded / approved / rejected / boss\_review / resumed / returned\_to\_requester / closed |
| actor\_id | uuid (FK → app\_user) | Null when the actor is the system (escalations, reminders, scheduled triggers) |
| attributed\_team\_id | uuid (FK → team) | The team a breach is charged to — the step's owning team; set on step\_kpi\_breach for team-level dashboard reporting (FR-31a). Blame is at team level, not individual |
| from\_step\_id | uuid (FK → step\_def) | Null unless the event moves the task |
| to\_step\_id | uuid (FK → step\_def) | Null unless the event moves the task |
| escalation\_level | int | Set on escalation events |
| detail | json | Comment, decision, SLA snapshot, reminder channel, etc. |
| created\_at | timestamp | Event time (UTC) |

Rows are never updated or deleted. The dashboard timeline, breach history, and time-per-step metrics all read from this table.

## **6\. Key Business Rules (testable)**

* **BR-1** A task advances to the next step only after the current step is approved.

* **BR-2** Each assignment's deadline is independent; escalation always grants a fresh response\_sla\_hours.

* **BR-3** Escalation within a step never skips order: PIC \#1 → PIC \#2 → … → boss.

* **BR-4** The step KPI is a hard cap: when total step business time exceeds step\_kpi\_hours, the task escalates up the management hierarchy — team boss, then boss of boss, then the company top escalation authority — independently of the silence track.

* **BR-4a** The step KPI clock starts when the task enters the step and is never **reset** while inside that step — not by silence-track hops, rejection, boss review, resume-in-place, or forwarding. It may be **paused** only while a creator info-request is pending, and resumes from where it stopped. Only a fresh step entry starts a new KPI clock.

* **BR-5** A rejection always routes to the boss before it can reach the requester.

* **BR-6** A boss override resumes the task at the rejected step, preserving prior approvals.

* **BR-7** A boss rejection returns the task to the requester.

* **BR-8** Worst-case time to clear a step ≈ (number of assignees in the fallback list) × response\_sla\_hours.

* **BR-9** Every state-changing action (create, assign, remind, escalate, approve, reject, boss review, resume, return, close) MUST write exactly one append-only audit\_event; no state change may occur without a corresponding audit record.

* **BR-10** A rejection MUST NOT be accepted without a non-empty comment. Comments and attachments are optional on approval.

* **BR-11** A request-for-information keeps the task on the same assignment at the same step and is not a rejection (does not route to the boss). While it is pending, the step KPI and the assignee's response SLA are **paused**; they resume only when the info is provided. The KPI is paused, never reset.

* **BR-11a** If the creator misses their info-request deadline, the request escalates to the creator's boss; if the creator's boss also takes no action, the request stays there (no further escalation). The step remains paused throughout, so the task parks until the info is provided.

* **BR-11b** The approver who raised an info-request may withdraw it at any time; on withdrawal the paused step clocks resume and the approver proceeds to decide with available information.

* **BR-12** The KPI escalation track terminates at the single company top escalation authority (team boss → boss of boss → top authority); it never climbs beyond that one person.

* **BR-13** A forward changes only the responsible person — the **step KPI is preserved unchanged**. A delegated assignment uses a **half-business-day response SLA**; if unanswered it escalates along the original step's chain (FR-16i). A delegate's action counts as the original assignee's, and the audit record names both the actor and the person acted for.

* **BR-14** While a standing delegation is active for a user (and in scope), assignments destined for that user are routed to the delegate instead; routing reverts automatically once the delegation ends.

* **BR-15** A task has exactly one active assignment at all times; escalation supersedes the current assignment rather than adding a parallel one.

* **BR-16** Escalation is monotonic up a single authority ladder (PIC \#1 → PIC \#2 → team boss → boss of boss → top authority). The task moves to max(current\_rung, proposed\_rung); an equal-or-lower trigger is logged but never reassigns or re-notifies, and the task never moves down.

* **BR-17** A step KPI breach MUST be attributed to the team that owns the step (attributed\_team\_id) and counted as that team's breach in the dashboard. Blame is at the team level; the holder at breach time remains traceable in the audit trail but is not the unit charged.

* **BR-18** A step contributes **at most one** breach to the dashboard, recorded at the first KPI/SLA miss within that step. Onward escalations (to the next person or the boss) triggered by that breach do not add further breaches.

## **7\. Non-Functional Notes**

* **Mobile-first notifications.** Push is the primary channel; design the notification service around it.

* **Auditability.** Every state change is persisted; the dashboard reads from audit\_event / task\_assignment, not from mutable task state alone.

* **Permissions.** Role-scoped visibility (member / boss / boss-of-boss / admin). Flow configuration is admin-only.

* **Timekeeping.** Store timestamps in UTC. SLA windows are measured in **business time** (see FR-16a): the deadline clock counts only configured working days/hours and pauses on weekends, holidays, and after hours. A shared **working calendar** service computes deadlines and elapsed business time consistently across escalation and reporting.

## **8\. Open Decisions**

All design decisions are resolved; the specification is build-ready. The only remaining inputs are **deployment configuration values**, not logic:

* **Working calendar** (FR-16a) — set the working hours per day (which defines a "business day" and therefore "half a business day"), the non-working weekdays, and the public-holiday list.

* **Default SLA/KPI values** per task type / step — response\_sla\_hours (default 24 business hours), step\_kpi\_hours, kpi\_escalation\_interval (default half a business day), delegated\_response\_sla (default half a business day), and the creator info-request SLA.

* **Top escalation authority** (FR-25a) — name the single company-wide person.

*End of specification.*