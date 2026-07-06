<!--
Sync Impact Report
==================
Version change: 1.3.1 → 1.3.2
Bump rationale: PATCH bump. Wording alignment only — no substantive change; no
  principle or standard added, removed, or redefined. Two edits make the document
  read consistently top-down with the Principle I audit boundary set in 1.3.1:
    1. Opening paragraph — "immutable history/audit" → "process-execution history"
       in the list of Flowable responsibilities. The business audit trail remains
       our own custom system of record (Principle I + Immutable Audit Trail standard).
    2. Technology & Platform Constraints, "Base engine" bullet — "history/audit" →
       "process-execution history", with a parenthetical noting the business audit
       trail is our own (see Principle I and the Immutable Audit Trail standard).

--- carried forward from 1.3.1 (PATCH) ---
Bump rationale: Clarifications and precision fixes only — no principle
  added, removed, or redefined; no section added or removed. Eight targeted edits:
    1. Principle I — history/audit over-claim fixed: Flowable owns process-execution
       history; our business audit trail is an acknowledged custom system of record
       (justified under Principle II), not a Flowable capability we must not build.
    2. Principle I — timer/SLA over-claim fixed: Flowable provides basic deadline
       timers (used for the per-person response SLA); the dual-clock KPI track
       (parallel step-KPI clock, max-wins authority ladder, pause-on-info-request,
       escalation dedup) is an acknowledged custom component under Principle II.
    3. Immutable Audit Trail (Platform Standards) — added a sentence noting it is our
       reporting system of record, distinct from but reconcilable with Flowable
       history (mirrors edit 1).
    4. Testing Standards — "BR-1..BR-18" replaced with "BR-*" so the count cannot go
       stale.
    5. Testing Standards (E2E) — Playwright (Java) kept as default; Selenium recorded
       as an explicit project-level allowance / documented override.
    6. Security Standards (No PII in logs) — example list made explicitly
       non-exhaustive.
    7. Security Standards (Attachment access control) — size/type limits reworded as
       per-configured-policy with defaults (50 MB, PDF/images/Office), aligning with
       Principle XIII instead of hard-coded constants.
    8. Principle X — added a clarification that revising/resubmitting a returned task
       is not approving it.
  Deliberately EXCLUDED: the "start from zero on edit/resubmit" behaviour — a
  spec-level decision to be settled in /speckit.specify, not the constitution.

--- carried forward from 1.3.0 (MINOR) ---
Bump rationale: Added five new core principles (IX–XIII), elevated two
  implied standards to first-class (Platform Standards), and corrected/expanded the
  business-hours timing rule. Additive only — all existing principles and sections
  kept and not renumbered; the timing change is a clarification/expansion, not a
  removal or redefinition. No breaking changes.

Modified principles:
  (added) → IX. Domain-Agnostic Platform
  (added) → X. Segregation of Duties
  (added) → XI. Minimise Customization by Joint Decision
  (added) → XII. Self-Hosted, Open-Source, No Per-Seat Fees
  (added) → XIII. Tunable Values Are Configuration, Not Code
  Principles I–VIII: unchanged (numbering preserved).

Corrected rule:
  - Technology & Platform Constraints — "Business-hours timing" replaced/expanded:
    business hours is the BASE unit; a business day is a derived multiple of
    configured working hours; a shared working calendar (hours/day, non-working
    weekdays, public holidays) governs and pauses all deadline calculations; the
    calendar is referenced by both the response SLA clock and the step KPI clock by
    default and is configurable per clock-type without code changes.

Added sections:
  - Platform Standards (elevated to first-class):
      • Mobile-First Notifications
      • Immutable Audit Trail

--- carried forward from 1.2.0 ---
Company (APPBAY STUDIO) standards imported and adapted to Java/Spring/Flowable:
  - Security Standards
  - API Design Standards
  - Testing Standards (reinforces Principles III & VIII)
  - Code Quality & Data Standards
  - Git & Documentation Standards
  - AI Agent Behaviour Rules
  - Project Overrides (company stack defaults do not apply; Java/Spring stack wins)

Stack adaptation notes (company JS/TS tool → this project's Java equivalent):
  - Bean Validation (Jakarta Validation) for input validation
  - JUnit 5 + Spring Boot Test for unit/integration
  - Testcontainers for real-database integration tests
  - Playwright (Java) / Selenium for E2E
  - Spring Data JPA / Hibernate for data access
  - Flyway / Liquibase for migrations
  - Spotless + Checkstyle for formatting/linting
  - BigDecimal for financial values
  - Company Node.js/TypeScript/React/Prisma defaults explicitly NOT imported.

Removed sections: none

Templates requiring updates:
  ✅ .specify/templates/plan-template.md — Constitution Check reads gates
     dynamically from this file; no hardcoded principle edits needed.
  ✅ .specify/templates/spec-template.md — scope/requirements align; no change.
  ✅ .specify/templates/tasks-template.md — test-first task ordering already
     compatible with Principles III & VIII and the new Testing Standards; no change.
  ✅ .specify/templates/checklist-template.md — generic; no change.

Follow-up TODOs: none.

--- Prior amendments ---
1.3.1 (2026-07-06): Eight clarifications/precision fixes — Principle I history/audit
  and timer/SLA over-claims corrected, Immutable Audit Trail distinguished from
  Flowable history, BR-* de-pinned, Selenium recorded as an explicit project
  allowance, PII list made non-exhaustive, attachment limits made per-policy, and
  Principle X clarified (revise/resubmit is not approval).
1.3.0 (2026-07-06): Added Principles IX–XIII, elevated Mobile-First Notifications and
  Immutable Audit Trail to first-class Platform Standards, and expanded the
  business-hours timing rule (base unit + shared/per-clock working calendar).
1.2.0 (2026-07-06): Imported APPBAY STUDIO company standards adapted to the
  Java/Spring/Flowable stack — Security, API Design, Testing, Code Quality & Data,
  Git & Documentation, AI Agent Behaviour Rules, and Project Overrides.
1.1.0 (2026-07-06): Added Principle VIII (Development Methodology: Superpowers) and
  named the base engine (Flowable) explicitly in Technology & Platform Constraints.
1.0.0 (2026-07-06): Initial ratification. First concrete constitution replacing
  the placeholder template; established Principles I–VII plus Technology &
  Platform Constraints and Development Workflow & Quality Gates sections.
-->

# EMIT Approval Workflow Engine Constitution

The EMIT Approval Workflow Engine is an approval-routing layer built **on top of**
Flowable (Java 17+, Spring Boot). Flowable is the process engine and is treated as a
trusted, unmodified dependency. We never reimplement or fork its core — process
execution, timers, human task handling, and process-execution history are Flowable's
responsibility. We build only the layer that is unique to us: our screens, our specific
business rules, our configuration, and our integrations.

## Core Principles

### I. Reuse Over Rebuild

If Flowable already provides a capability, we MUST use it rather than write our own.
This explicitly includes step execution and waiting on a human (user tasks). Custom code
MUST exist only for what Flowable does not provide. Reimplementing, forking, or shadowing a
Flowable core capability is prohibited.

Two boundaries are drawn precisely so this principle is not over-claimed:

- **History/audit**: Flowable owns PROCESS-EXECUTION history — its own engine-level history
  tables for how a process instance executed. Our business-level audit/reporting layer (the
  Immutable Audit Trail standard) is a SEPARATE, custom system of record. It MAY read from
  Flowable history but is authoritative for dashboards, breach metrics, team attribution, and
  business events. That business audit trail is an acknowledged custom component justified
  under Principle II — not something we must not build.
- **Timers/SLA**: Flowable provides basic deadline/timer capability, which we use for the
  per-person silence-track response SLA. The dual-clock KPI track — the parallel step-KPI
  clock, the single max-wins authority ladder, pause-on-info-request, and escalation
  deduplication — is NOT a stock Flowable capability. It is an acknowledged custom component
  justified under Principle II. This principle does not imply all SLA/timer behaviour is
  Flowable's.

Rationale: Flowable's engine is already correct, tested, and audited. Re-solving those
problems adds risk and cost with no benefit — but the boundary must be stated accurately so
genuinely custom capabilities are recognised as such rather than assumed to be stock.

### II. Thin Custom Layer

The custom layer MUST be kept as small as possible. Every custom component MUST justify,
in writing, why Flowable cannot provide the capability before it is built. When a
requirement can be met by Flowable configuration or a thin adapter, that path MUST be
chosen over new custom code.

Rationale: The smaller the custom code, the smaller the security review and the lower the
maintenance burden. Custom surface area is a liability to be minimized, not a default.

### III. Test-First (NON-NEGOTIABLE)

Every business rule MUST be expressed as a failing test before any implementation code is
written. The specification's business rules are the source of truth for these tests.
Red-Green-Refactor is enforced: write the test, watch it fail, then implement. No business
rule ships without a corresponding test.

Rationale: Tests derived from the spec are the objective, executable definition of
"correct." Writing them first prevents implementation bias and guarantees coverage of every
agreed rule.

### IV. Spec Is The Boundary

Nothing is built that is not in the agreed specification. New requests are changes to the
spec first, then code — never code bolted on ahead of the spec. Any behaviour absent from
the spec is out of scope until the spec is amended.

Rationale: The spec is the single agreed boundary of the system. Treating it as the entry
point for all change prevents scope creep and keeps the control surface auditable.

### V. Behaviour Over Implementation

Correctness MUST be judged by passing tests and working demos, not by inspecting source
code. The plain-English specification and its tests are the control surface. Reviews and
acceptance decisions are made against observable behaviour.

Rationale: Behaviour is what the business depends on. Anchoring correctness to observable
behaviour keeps the spec and tests — not internal code structure — as the authority.

### VI. Configuration, Not Code, For Flows

Task types, steps, teams, and assignees are data/configuration, not code. Adding,
removing, or reordering steps MUST NOT require a code change or redeployment of custom
code. Flow shape is expressed as configuration (e.g. Flowable process definitions and our
configuration data).

Rationale: Approval flows change often and are owned by the business. Making them data lets
flows evolve without engineering work and without re-triggering security review of code.

### VII. One System, No New Silos

This engine routes approvals only. It MUST NOT store inventory, vendors, or other domain
data owned by other systems. It integrates with those systems via webhooks rather than
duplicating their data. Domain data is referenced, never copied into this engine as a
system of record.

Rationale: Duplicating another system's data creates a new silo that drifts out of sync and
expands the security and compliance footprint. Integration over duplication keeps this
engine focused and authoritative only for approval routing.

### VIII. Development Methodology: Superpowers

All development MUST be carried out using the Superpowers agentic methodology
(https://github.com/obra/superpowers). Concretely:

- **Strict red-green TDD**: write a failing test, confirm it fails, write the minimal code
  to make it pass, then refactor. This is the mandatory rhythm for every change.
- **Small, reviewed tasks**: work MUST be broken into small tasks. Each task is reviewed for
  spec-compliance and then for code-quality before the next task begins.
- **YAGNI and DRY**: no speculative features and no duplicated logic. Build only what a
  current task requires; factor out duplication when it appears.

This principle is the **enforcement mechanism** for Principle III (Test-First) and
Principle V (Behaviour Over Implementation) — it defines HOW those are practised, not a
separate requirement. Division of tools: Spec Kit governs the artifacts (constitution, spec,
plan, tasks); Superpowers governs the coding behaviour. Their authority does not overlap.

Rationale: Principles III and V state WHAT correctness means; without a defined practice,
"how" is left to chance. Superpowers supplies the disciplined, repeatable practice — strict
TDD, small reviewed increments, YAGNI/DRY — that makes those principles operational.

### IX. Domain-Agnostic Platform

The engine MUST remain generic. No domain-specific or department-specific logic may be
hard-coded — nothing that only makes sense for Finance, inventory, claims, or any single
department. A task type is neutral configuration on a generic core; the engine MUST NOT
"know" what kind of task it is running.

Rationale: The client requires a platform that handles any task type, not a digitised
point-solution over-fitted to one department.

### X. Segregation of Duties

The user who raises a task MUST NOT be an approver of their own task at any step. This
control holds across every flow and MUST be enforced server-side. It is non-negotiable.

Clarification: receiving a task back to REVISE and resubmit (e.g. after a rejection is
upheld) is NOT the same as approving it. The raiser-cannot-approve rule prohibits the
requester acting as an approver of their own task — not the requester revising their own
returned task.

Rationale: Approval must always come from someone other than the requester.

### XI. Minimise Customization by Joint Decision

Flowable capability and configuration MUST be preferred over custom code. Any net-new
custom scope MUST be a decision made jointly with the client, never added unilaterally.

Rationale: Prevents scope creep / "perpetual development" and keeps the security-review
surface small. Reinforces Principles I and II.

### XII. Self-Hosted, Open-Source, No Per-Seat Fees

The system MUST be deployable on the organisation's own infrastructure (self-hosted).
Open-source components MUST be preferred, and per-user/per-seat licensing MUST be avoided.

Rationale: A client requirement for control, cost, and no vendor lock-in. This supersedes
the company constitution's cloud/AWS defaults for this project (see Project Overrides).

### XIII. Tunable Values Are Configuration, Not Code

All operational thresholds — response SLA, step KPI, reminder lead time, delegation window,
KPI escalation interval, and the top escalation authority — MUST be configurable per
process/task type. They MUST NOT be hard-coded.

Rationale: The client requires per-process tunability without rebuilds. Extends Principle VI
(Configuration, Not Code, For Flows) to all operational thresholds.

## Technology & Platform Constraints

- **Stack**: Java 17+, Spring Boot, and the Flowable engine. A custom REST API and web
  frontend form the interaction layer.
- **Base engine**: Built upon Flowable (https://github.com/flowable/flowable-engine), used
  as an unmodified, trusted dependency for the process engine, timers, human tasks, and
  process-execution history (the business audit trail is our own — see Principle I and the
  Immutable Audit Trail standard).
- **Time storage**: All timestamps MUST be stored in UTC.
- **Business-hours timing (base unit)**: All SLA and KPI timing MUST be measured in BUSINESS
  HOURS as the base unit. A business day is a derived multiple of the configured working
  hours (e.g. 8 business hours = 1 day, 4 = half a day). Day- and half-day intervals MUST be
  computed from hours, never stored as the base unit. Wall-clock elapsed time MUST NOT be
  used for SLA/KPI measurement.
- **Shared working calendar**: A shared working calendar (working hours per day, non-working
  weekdays, public-holiday list) MUST govern all elapsed-time and deadline calculations, and
  MUST pause outside working hours, on weekends, and on public holidays.
- **Per-clock calendar configuration**: By default the calendar is referenced by BOTH the
  per-assignment response SLA clock and the step KPI clock, so no one is penalised for time
  the office was closed. The calendar MUST be configurable per clock-type, so a different
  calendar can later be applied to the KPI clock versus the personal clock without code
  changes.
- **Integration boundary**: External systems are reached via webhooks (see Principle VII);
  their data is not persisted as a system of record here.

## Development Workflow & Quality Gates

- **Spec-first change flow**: All change begins with a spec amendment (Principle IV). Plans
  and tasks derive from the spec; code derives from tasks.
- **Test-first gate**: A business rule may not be implemented until a failing test for it
  exists (Principle III). Reviews confirm the test existed and failed before implementation.
- **Custom-code justification gate**: Any new custom component MUST carry a written
  justification of why Flowable cannot provide the capability (Principles I & II). Absent
  that justification, the component is rejected.
- **Configuration gate**: Changes to flow shape (task types, steps, teams, assignees) MUST
  be delivered as configuration, not code (Principle VI). A flow change that requires a code
  change is a design defect to be corrected.
- **Acceptance basis**: Features are accepted on passing tests and working demos, not source
  inspection (Principle V).

## Platform Standards

### Mobile-First Notifications

Mobile push MUST be the primary notification channel, and the notification service MUST be
designed around it. Users MUST be notified on assignment, reminder, and escalation.

Rationale: The primary user touchpoint is mobile; other channels are secondary to push.

### Immutable Audit Trail

Every state-changing action MUST write exactly one append-only audit record capturing actor
(or "system"), timestamp, event type, from/to state, and relevant detail. Audit records MUST
NEVER be updated or deleted. The dashboard and reporting MUST read from this trail. This
business audit trail is our system of record for reporting and MAY be fed by or reconciled
with Flowable's process-execution history, but is distinct from it (see Principle I).

Rationale: Tamper-evident history is a core trust and compliance requirement. This makes the
history/audit reliance (Principles I & VII) and state-transition logging (Code Quality & Data
Standards) first-class and non-negotiable.

## Security Standards

- **RBAC on protected endpoints**: All protected endpoints MUST enforce role-based access
  control server-side. Client-provided roles MUST NEVER be trusted; roles are derived from
  the authenticated session/token.
- **Input validation at the boundary**: All input MUST be validated and sanitised at the API
  boundary using Bean Validation (Jakarta Validation, `jakarta.validation.*`).
- **No PII in logs**: PII MUST NEVER be logged — e.g. IC numbers, passport numbers, full card
  numbers, and any other personally identifiable information (the list is non-exhaustive).
- **Secrets management**: Secrets MUST live in environment variables or a secrets manager and
  MUST NEVER be committed to git.
- **CORS**: An explicit origin whitelist MUST be configured; wildcard origins MUST NOT be
  used in production.
- **Rate limiting**: All public-facing endpoints MUST be rate limited, INCLUDING incoming
  webhooks (the SAP/external integration surface).
- **Attachment access control**: Attachments MUST be access-controlled to the same users who
  may view the task. Attachment constraints (accepted file types and per-file size) MUST be
  enforced per configured policy — defaults: 50 MB per file and accepted types PDF, images,
  and Office documents — rather than hard-coded as fixed constants (see Principle XIII).
- **Dependency vulnerabilities**: The build MUST fail on high/critical dependency
  vulnerabilities unless explicitly justified.

## API Design Standards

- **Resource-based URLs**: URLs MUST be RESTful and resource-based (`/api/v1/tasks`, not
  `/api/v1/getTasks`) with semantic HTTP verbs (GET/POST/PUT/PATCH/DELETE).
- **Consistent response envelope**: Responses MUST use a consistent envelope carrying a
  success flag, a data payload, a human-readable message, and a machine-readable error code.
- **Pagination**: All list endpoints MUST be paginated — never unbounded.
- **Versioning**: APIs MUST be versioned via URL prefix (`/api/v1/`); breaking changes MUST
  introduce a new version.
- **Documentation**: All endpoints MUST be documented via OpenAPI/Swagger.

## Testing Standards

Reinforces Principle III (Test-First) and Principle VIII (Superpowers TDD methodology).

- **Coverage of business rules**: Unit and integration tests are REQUIRED for all business
  rules, written test-first (before implementation).
- **Unit/integration framework**: Unit and integration tests MUST use JUnit 5 with Spring
  Boot Test.
- **Real database in integration tests**: Integration tests MUST run against a real database
  via Testcontainers — the database MUST NOT be mocked.
- **End-to-end tests**: Key user flows (raise, approve, reject, request-info, escalation)
  MUST have end-to-end tests. Playwright (Java) is the default E2E framework. Selenium is
  permitted for THIS project as an explicit project-level allowance — a documented override
  of the company Playwright-only standard, not a silent contradiction.
- **Coverage target**: Target ~90% coverage on custom code. The spec's business rules (BR-*)
  are the source of truth for tests.
- **Test placement**: Test files MUST live adjacent to the code they test.

## Code Quality & Data Standards

- **Data access**: Database access MUST go through Spring Data JPA / Hibernate. N+1 queries
  MUST be avoided using fetch joins / entity graphs.
- **Schema migrations**: Schema changes MUST be delivered as versioned migrations using
  Flyway or Liquibase — never hand-edited SQL on production.
- **Formatting/linting**: Formatting and linting MUST be enforced via Spotless + Checkstyle,
  with configuration committed to the repo.
- **Package structure**: Java packages (e.g. `com.emit.approval.*`) MUST be organised by
  feature/domain.
- **Financial arithmetic**: Money and any financial values MUST use decimal arithmetic
  (`BigDecimal`) — never floating point.
- **State-transition logging**: Every state transition MUST log actor, timestamp, and
  previous state (covered by the audit trail).
- **No hard-delete**: Business records (tasks, assignments, audit) MUST use soft-delete /
  status-change only — never hard-delete. A cancelled task is a status, not a deletion.

## Git & Documentation Standards

- **Commits**: Conventional Commits MUST be used (`feat:`, `fix:`, `chore:`, `docs:`,
  `refactor:`, `test:`).
- **Branch naming**: Branches MUST follow `feature/<name>`, `fix/<name>`, `hotfix/<name>`.
- **Pull requests**: PRs MUST reference a spec item; include what/why/how-to-test; and be
  kept small and focused.
- **Documentation artifacts**: The repo MUST maintain a README, a CHANGELOG (Keep a Changelog
  format), and Architecture Decision Records (ADRs) in `docs/adr/` for major technical
  decisions — especially any decision to build custom rather than use Flowable
  (see Principles I & II).

## AI Agent Behaviour Rules

- The agent MUST read this constitution at the start of every session before writing code.
- The agent MUST NOT add dependencies without listing and justifying them in the plan.
- The agent MUST NOT modify this constitution unless explicitly instructed.
- The agent MUST NOT hard-code credentials, environment-specific URLs, or secrets.
- The agent MUST ask for clarification rather than assume when a requirement is ambiguous.
- The agent MUST generate tests alongside implementation.
- The agent MUST flag any conflict between the spec and this constitution before proceeding.

## Project Overrides

- **Stack authority**: The EMIT Approval Workflow Engine uses Java 17 + Spring Boot +
  Flowable per this project constitution. The company (APPBAY STUDIO) Node.js / TypeScript /
  React / Prisma stack defaults do NOT apply to this project. Reason: this project is built on
  the Flowable BPM engine (JVM). All company standards that are stack-agnostic (security, API
  design, testing discipline, git, documentation, AI behaviour) still apply, translated to the
  Java/Spring stack above.

## Governance

This constitution supersedes other development practices for the EMIT Approval Workflow
Engine. Where a practice conflicts with a principle here, the principle wins.

- **Amendments**: Changes to this constitution MUST be documented (what changed and why),
  reviewed, and approved before taking effect. Amendments that alter flow, spec, or testing
  discipline MUST include a migration note describing impact on existing specs and tests.
- **Versioning policy**: This constitution is versioned using semantic versioning.
  MAJOR = backward-incompatible governance or principle removals/redefinitions;
  MINOR = a new principle/section or materially expanded guidance;
  PATCH = clarifications and non-semantic refinements.
- **Compliance review**: Every plan, spec, and review MUST verify compliance with these
  principles. Violations MUST be either corrected or explicitly justified in a Complexity
  Tracking record before work proceeds. Unjustified custom code, out-of-spec work, or
  untested business rules MUST NOT be merged.

**Version**: 1.3.2 | **Ratified**: 2026-07-06 | **Last Amended**: 2026-07-06
