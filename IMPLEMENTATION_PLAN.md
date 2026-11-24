# Tequity two-way sync: phased execution plan

## How to use this plan

In a new Codex chat opened in this repository, say **"proceed with phase X"**. `AGENTS.md` directs Codex to this file. Execute the requested phase, check dependencies, and update the status and execution log. This file is the persistent project reference; a new chat should not depend on access to earlier conversations.

Source of truth: `Round1-Tequity.html`. This plan proposes work; it does not mean the design decisions have already been validated or implementation is authorized.

## Problem and scope

Design our side of bidirectional customer/contact synchronization between Salesforce and an operations portal backed by PostgreSQL. Salesforce calls our HTTPS webhook endpoint; our worker calls the Salesforce create/update record API. Both sides create and edit. Preserve changes to different fields, choose a defensible current value for same-field conflicts, and retain all distinct edits with actor and time.

Production requirements: 5 million combined records, visibility within 30 seconds while both systems are available, duplicate and out-of-order webhook handling, no sync loops, at most 100 outbound Salesforce API requests/second including retries, and durable recovery from a six-hour outage with potentially missing webhooks.

Required: design document, authentic unedited AI chat history, repository with all commits, open questions, and a 10-minute walkthrough. Optional: a TypeScript/Node.js 20+ demo against a fake CRM. Exclude deletes, visual UI, Salesforce login screens, second region, real Salesforce load tests, and production service hardening beyond what is necessary to explain correctness.

## Recommendations carried forward

1. **Recover history through a durable replayable change history.** Assume an upstream interface can replay every distinct committed change after a saved checkpoint, with event identity, changed values, actor, time, and useful ordering metadata. Retention must exceed six hours plus recovery and safety margin; choose and justify a concrete retention assumption in Phase 1. This is not a capability guaranteed by the brief or verified against real Salesforce.
2. Advance a recovery checkpoint only with the transaction that commits the associated changes and audit history. Define a safe contiguous checkpoint rule if events are processed concurrently. Replay and webhooks share deduplication and conflict handling.
3. A modified-record snapshot query can repair current state but cannot reconstruct intermediate edits and actors. If replay/audit access is unavailable, explicitly identify the unmet history requirement and the necessary integration or requirement change. Do not pretend polling resolves it.
4. Use durable outbound work and one shared rate budget for every outbound request, including writes, retries, reads, recovery, and conflict repairs. Honor throttling and retry delays without discarding work.
5. Provisional capacity example: 20 portal edits/second, one outbound write per edit, six-hour sustained outage, and a 90 requests/second operating ceiling. Backlog = `20 × 21,600 = 432,000`. With ongoing writes at 20 requests/second and recovery/retry allowance at 10 requests/second, net drain = `90 - 20 - 10 = 60`; time = `432,000 / 60 / 60 = 120 minutes`. These are assumptions, not a measured SLA. Phase 1 must also state Salesforce-origin and quiet-hour rates. Phase 6 must calculate inbound replay volume, pagination, repairs, request amplification, and storage separately; revise this example if those costs exceed the allowance.
   Phase 6 result: the one-call example above is superseded by `docs/recovery-capacity.md`. Three calls per mapped update, 89/s achieved service within 90/s pacing, 10/s exceptional overhead and a 73-minute replay barrier yield approximately 1,440.37 minutes (24 hours) recovery. Worst-case healthy burst latency remains an explicit feasibility gate; see the Phase 6 timing assumptions.
6. Schedule fresh traffic and backlog so neither starves. Explicitly define whether the 30-second target resumes during catch-up and what assumptions make it feasible. Do not promise 30 seconds under unbounded arrivals, delays, throttling, or failures.
7. Assume an upstream idempotency or unique external-ID mechanism for safe retried creates only if it is explicitly documented in the design. Handle requests accepted remotely whose response is lost.
8. Prefer a single application with HTTP and worker roles, separated by durable database state, unless a concrete subproblem justifies another component. Keep the demo runnable without external services; distinguish its local persistence from production PostgreSQL.
9. Preserve every edit in history even if pending writes are combined. Do not enable coalescing until its effects on history, intermediate remote values, conflict ordering, and demo expectations are defined.

## Phase status

| Phase | Name | Dependencies | Status |
|---|---|---|---|
| 1 | Assumptions and feasibility | None | Complete |
| 2 | Conflict and history rules | 1 | Complete |
| 3 | Durable data and boundaries | 1–2 | Complete |
| 4 | Inbound synchronization | 1–3 | Complete |
| 5 | Outbound synchronization | 1–4 | Complete |
| 6 | Outage recovery and capacity | 1–5 | Complete |
| 7 | Required submission | 1–6 | Complete — remaining submission items resolved by user confirmation |
| 8 | Optional executable demo | 1–7 | Not started |

Phases 1–6 are design work first. Phase 8 implements the agreed design. Updating earlier decisions when evidence changes is allowed, with the reason recorded.

## Phase 1 — Establish assumptions and feasibility

Work:
- Read the original brief and create a requirements-to-validation checklist.
- State peak and quiet edit rates for each direction, burst behavior, request amplification, and webhook delivery assumptions needed for 30 seconds.
- Choose changed-field payloads as the provisional contract; explain the full-record alternative and why omitted, null, unchanged, and cleared values must differ.
- Define trusted portal actor/time sources, source ordering metadata, clock uncertainty, identity mapping, owner mapping, and parent/customer dependencies.
- Specify the required replay/audit contract and retention, plus idempotent creation and conditional-write capabilities needed for correctness. Clearly separate assumptions from verified capabilities.
- For each open question, explain what changes if its answer differs. Include existing-data bootstrap for 5 million records as a scope decision.

Artifacts: `docs/assumptions.md`, `docs/requirements-checklist.md`.

Acceptance: every material unknown has a question, provisional answer, and consequence; infeasible guarantees are identified. The capacity model includes both edit directions and quiet traffic.

## Phase 2 — Define conflict and history rules

Work:
- Define per-field version comparison using named metadata, deterministic ties, timestamp limitations, and same-side concurrency behavior.
- Obtain portal actor from authenticated identity and assign authoritative time/version on the server; do not trust the sample's user-supplied actor blindly.
- Preserve all distinct edits, including losing and late edits; distinguish duplicate delivery from a distinct edit with the same value.
- Explain what concurrent portal users see, what Salesforce receives, and how both systems converge after opposite-side conflicts.
- Account for remote writes racing with local decisions: identify required conditional writes, read/merge/retry or correction behavior and their request cost. Simply suppressing every inbound-triggered write may prevent convergence; distinguish echoes from necessary repairs.
- Walk through different-field edits, same-field conflicts, ties, out-of-order changes, clock skew, and API acceptance with a lost response.

Artifact: `docs/conflict-rules.md`, with worked examples and expected current/history results.

Acceptance: rules are deterministic, different-field edits survive, losing edits remain in history, and both replicas can converge under the stated upstream assumptions.

## Phase 3 — Design durable data and boundaries

Work:
- Define production PostgreSQL tables for customers, contacts, source IDs, field versions, audit entries, received event IDs, outbound jobs, and replay checkpoints.
- Define unique constraints, indexes, transactional writes, per-record concurrency control, worker claim/reclaim behavior, and crash boundaries.
- Diagram callers, HTTPS network boundary, HTTP role, worker role, and durable state crossing their boundary.
- Address create mapping, customer-before-contact dependencies, lookup scaling, history growth, and retention at five million records.
- Specify webhook validation/application/acknowledgment semantics in light of the brief's wording; avoid acknowledging an uncommitted change.

Artifacts: `docs/data-model.md`, `docs/architecture.md` with diagram and interface sketches.

Acceptance: each component solves a named subproblem; transactions and crash recovery are explicit; model supports prior phase rules.

## Phase 4 — Specify the inbound path

Work:
- Define authentication, schema validation, event identity, object/field allowlists, and error responses.
- Atomically deduplicate, append audit history, apply field decisions, and commit before acknowledging.
- Define late events, unknown records, dependent contacts, replay overlap, invalid events, and full-record payload handling.
- Distinguish source echoes from genuine remote changes and identify when convergence needs a repair job.

Artifact: `docs/inbound-flow.md` with pseudocode and scenario outcomes.

Acceptance: a duplicate has no extra effect; an older edit remains audited without overwriting newer fields; retries after a crash are safe; normal propagation fits a conditional latency budget.

## Phase 5 — Specify the outbound path

Work:
- Atomically persist portal values, field metadata, history, and outbound jobs.
- Define field patches, ordering per record, dependency handling, request identity, conditional updates, and acknowledgment state.
- Enforce a strict configured outbound cap without unintended bursts, counting retries and all worker roles together.
- Define timeouts, backoff with jitter, retry delays, permanent failures, throttling, and ambiguous successful creates/updates.
- Define loop prevention using provenance/correlation assumptions and version semantics; avoid treating every integration-user event as an echo.

Artifact: `docs/outbound-flow.md` with job state machine and failure cases.

Acceptance: pending work survives restarts; accepted-but-unacknowledged requests recover safely; retries obey the cap; echoes stop without preventing genuine conflict repair.

## Phase 6 — Design outage recovery and capacity

Work:
- Describe disk state throughout a six-hour outage and every restart boundary.
- Replay missing inbound history from a committed checkpoint with overlap/deduplication and an explicit replay-completion boundary.
- Define coordination between replay and outbound backlog so stale queued writes do not erase unseen remote edits.
- Calculate outbound backlog, inbound event backlog, recovery pages, repair/retry cost, storage needs, and drain times with ongoing traffic. Show division in minutes and identify the bottleneck.
- Use the 120-minute example only if the full request accounting supports it. Explain how throttling and increased rates change the result.
- Define scheduling and the normal-operation latency budget, plus failure behavior when assumptions no longer hold.

Artifact: `docs/recovery-capacity.md`.

Acceptance: no history-recovery claim depends on snapshots alone; all calls fit the shared rate budget; calculations distinguish estimates from guarantees; backlog eventually drains under stated load.

## Phase 7 — Assemble the required submission

Work:
- Consolidate the design into `docs/design.md` following all 12 sections requested in the HTML, with assumptions first.
- Select exactly three primary operational metrics and concrete escalation thresholds, with reasoning.
- Prepare `docs/walkthrough.md` for a 10-minute presentation and `docs/submission-checklist.md` for artifacts and remaining gaps.
- Review duplicate, older-event, loop, and six-hour-outage rules end to end for contradictions.
- Explain the authentic chat export process available to the user; mark the export pending until actually supplied. Inspect repository/commit availability and preserve history without inventing work or retroactive conversations.

Artifacts: consolidated design, walkthrough, submission checklist; authentic chat export only if available.

Acceptance: design is buildable, all brief sections are addressed, assumptions and alternatives are visible, and required unavailable submission items are honestly marked pending. Mark this phase complete only when its required items are satisfied; otherwise record partial completion and the exact gap.

## Phase 8 — Build and validate the optional demo

Work:
- Implement TypeScript on Node.js 20+ with a fake CRM and durable local persistence, mapping to production design concepts. Choose dependencies compatible with the agreed Node minimum and Node-only execution constraint.
- Provide one command after `npm install` to run a deterministic narrated demo from a fresh checkout.
- Verify duplicate delivery, late events, field conflicts, loop suppression and repairs, parent/child creates, and rate enforcement.
- Demonstrate a real process restart while the fake CRM is unavailable, durable pending calls, and replay of changes whose webhook was dropped. Use simulated time for six-hour arithmetic; label simulation honestly.
- Check ambiguous responses and prove the cap counts retries. A lowered configurable cap may make behavior visible.
- Add `README.md` with execution instructions, limitations, and production/demo differences. Reconcile implementation against design and record any justified changes in both.

Artifacts: source, meaningful automated checks, demo command, README, validation results.

Acceptance: fresh-checkout demo requires only `npm install` plus one command and no external services; all four brief scenarios pass; durable restart and configured cap are demonstrably enforced; design and code agree.

## Execution log

| Entry | Work | Validation / outstanding items |
|---|---|---|
| 24 November 2026 — Phase 7 closure | User confirmed that all three outstanding items have been taken care of: complete authentic chat history, final repository packaging/access, and revised submission timing/delivery. Marked Phase 7 complete and reconciled current submission status in the design and checklists. | External completion is recorded on the user's confirmation, not an independent assistant inspection of exports or delivery. Earlier partial-status entries remain historical. Production Salesforce capability assumptions and the healthy worst-case 30-second feasibility gate remain open. No chat edits, git commits or external messages performed; Phase 8 remains unstarted. |
| 24 November 2026 — Phase 7 | Reused Phases 1–6 and read the original brief; created `docs/design.md` with all 12 requested sections, exactly three primary metrics and escalation thresholds, `docs/walkthrough.md` with a 600-second presentation allocation, and `docs/submission-checklist.md` with export guidance, commit inventory, preservation baseline and end-to-end scenario review. Updated requirements validation with current evidence. | Partial: design assembly/review completed, but full authentic chat provenance/coverage remains pending (existing file preserved, not certified complete); final authorized repository packaging/access validation and revised deadline/delivery remain pending. Reviewed duplicate/older/echo-repair/outage invariants, local links, arithmetic, metric/section counts and diff whitespace. Three existing commits preserved at HEAD `9b6d404`; no commit or external submission made. About 1,440.37-minute recovery supersedes 120 minutes; worst-case healthy 30-second burst feasibility remains explicitly unproven. No application tests, real Salesforce verification or Phase 8 implementation. |
| 24 November 2026 — Phase 6 | Created `docs/recovery-capacity.md`: six-hour durable state, account outage probing, recovery epochs/partition high-water proofs, bounded overlap/manifests/contiguous checkpoints, global replay barrier and per-record history-covered conditional release, restart table, full peak/quiet request and echo budgets, fair scheduling, latency limitations, sensitivity, connection sizing, storage and retention. Updated `docs/assumptions.md` to supersede preliminary estimates. | Checked arithmetic using Python, manually traced crash/overlap/conflict/ambiguity/dependency cases against Phases 2–5 and the brief, and ran diff whitespace checks. Design acceptance met under explicit upstream/load/service assumptions: replay does not rely on snapshots, calls share the cap, estimates are conditional, and 19/3 old jobs/s net service is positive. Recovery estimate is 1,440.37 minutes, not 120; peak storage allowance is about 16.77 GiB/day logical growth. Healthy 30-second worst-case burst scheduling remains unproven and explicitly gated on stronger timing/measurement or contract changes. Q03/Q04/Q08–Q15/Q17 remain provisional. No executable tests, production benchmark, real Salesforce verification, code, chat-history edits or git commits; Phases 7–8 remain unstarted. |
| 24 November 2026 — Phase 5 | Created `docs/outbound-flow.md`: atomic portal save/outbox, per-field job outcomes and state machine, per-record dispatch/dependencies, revision/history-covered conditional patches, non-destructive create-or-resolve, generation-safe acknowledgment, strict shared rolling-second pacing, fenced egress failover, retry/throttle policy, ambiguous mutation reconciliation, and provenance-based echo/repair rules. Refined the shared-budget assumption with sole egress ownership and conservative failover; no later phase implemented. | Manually traced Phase 5 acceptance, Phase 2 A–N, Phase 3 crash/lease boundaries, and Phase 4 repair reuse; checked rolling-window spacing and stale-dispatch hazards. Design acceptance met under explicit upstream and fenced-egress assumptions. Conditional outbound allocation totals 25 seconds only with bounded coverage/call paths; Phase 6 must validate amplification, capacity, recovery barriers and scheduling. No executable tests, production measurement, or real Salesforce verification claimed. Q03/Q04/Q08–Q14/Q17/Q18 remain provisional. Existing repository history preserved, including concurrently observed commit `87d273d`; no commit created by this phase. Existing `Chat_History/` left untouched; authenticity/completeness not assessed here and submission validation remains Phase 7. Phases 6–8 remain unstarted. |
| 24 November 2026 — Phase 4 | Created `docs/inbound-flow.md`: provisional signed HTTPS contract, strict envelope/object/field validation, event/edit identity and immutable-content checks, atomic apply-before-acknowledge pseudocode, response/remediation matrix, placeholder/dependency hydration, full-record restrictions, per-field echo/repair classification, replay overlap, and 16 scenario outcomes. Refined Q13 with a 300-second delivery-signature window (independent of edit time), Q18 with explicit provisional field limits/nullability, and repair-intent reuse when new remote evidence invalidates earlier satisfaction. | Manually traced Phase 4 acceptance, relevant Phase 2 A–N cases, Phase 3 lock/uniqueness/crash boundaries, mixed events, dependency hydration, and contiguous replay completion. Conditional healthy inbound budget totals 20 seconds with 10 seconds margin; no measured latency, executable tests, live endpoint, or Salesforce capability verification claimed. Design acceptance met under existing assumptions; auth/schema/retry, replay/history/coverage, provenance, mapping, and durability gates remain open. Authentic chat export pending. Existing working changes/history preserved; no code or commit created. Phases 5–8 remain unstarted. |
| 24 November 2026 — Phase 3 | Created `docs/data-model.md` and `docs/architecture.md`: production table/constraint/index specification, atomic save/inbound boundaries, entity locks, worker leases and ambiguous-mutation recovery, replay manifests/contiguous checkpoints, parent/owner dependencies, scale inputs, caller diagram, and interface sketches. Clarified portal time as database server time at the serialized decision boundary rather than an unavailable exact WAL commit timestamp. | Reviewed Phase 2 A–N against persistence invariants and crash boundaries; checked component purpose, HTTPS callers, durable role boundary, field counts and history arithmetic. Design acceptance met under existing upstream assumptions; no migration, executable tests, production benchmark, or Salesforce verification claimed. Replay/provenance/idempotency/conditional-write gates, retention policy, and dependency contract remain open. Authentic chat export pending. Existing working changes/history preserved; no commit created. Phases 4–8 remain unstarted. |
| Plan creation | Saved phases, recovery/capacity recommendations, and new-chat routing instructions. No implementation phases started. | Original HTML read in the planning chat. Real Salesforce capabilities remain unverified assumptions. |
| 24 November 2026 — Phase 1 | Read full original article, plan, instructions, and repository state. Created `docs/assumptions.md` and `docs/requirements-checklist.md`; selected provisional patch contract, both-direction workloads, 72-hour replay retention, identity/time sources, bootstrap scope, and explicit upstream feasibility gates. | Reviewed Phase 1 work/acceptance and mapped all brief requirements and 12 design sections to validation. Peak 20 edits/s and quiet 1 edit/s per direction; six-hour volumes 432,000 each. 120-minute outbound estimate remains conditional; full recovery accounting belongs to Phase 6. Q01–Q20 retain client questions and consequences, especially replay, conditional writes, idempotency, latency bounds, historical bootstrap, and overdue submission timing. Authentic chat export pending. Existing commit `4aa91fb` preserved; no new commit authorized or created. Phases 2–8 not started. |
| 24 November 2026 — Phase 2 | Created `docs/conflict-rules.md` with per-source field candidates, deterministic timestamp/rank arbitration, trusted portal saves, immutable edit history, echo/repair distinction, conditional-write reconciliation, and 14 worked cases A–N. Updated assumptions with revision-associated history coverage, request-cost caveat, stale-work suppression semantics, and Q21 on history location. | Manually traced expected current/history outcomes, late/different-field edits, ties, clock rollback, remote races, and lost create/update responses; no executable tests or Salesforce verification claimed. Design meets Phase 2 acceptance under explicit upstream assumptions. Client policy/provenance/history capabilities remain open; Phase 6 must revise amplification if coverage reads exceed allowance. No code or commits created; Phases 3–8 remain unstarted. |
