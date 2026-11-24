# Requirements-to-validation checklist

Source: `Round1-Tequity.html`, read in full on 24 November 2026. Status records evidence available now; a future check is not a passed check. Q references point to `assumptions.md`. Phases 1–6 produce design evidence; optional demo tests cannot verify real Salesforce capabilities or production capacity.

| ID | Requirement / source section | Validation and acceptance evidence | Phase / current status |
|---|---|---|---|
| R01 | Assumptions and open questions first; explain alternatives | Every material unknown has question, provisional answer, consequence. | 1: covered by Q01–Q20; client answers pending. |
| R02 | Explicit field scope, customers/contacts, 5 million combined | Check schema and mappings against explicit field list; indexed lookups and storage/capacity model. | 1: scope recorded; 3,6: sizing pending. |
| R03 | Both sides create and update | Trace create/update each direction, unique mapping and parent dependency; ambiguous create response scenario. | 1: Q08,Q11,Q12; 3–5 pending. |
| R04 | Worst-case 30 s while healthy; everyday sooner | Bound source delay, queue, processing, API and visibility; prove bounded workload fits; distinguish catch-up. | 1: conditional <=25 s envelope and limits stated; 4–6 proof pending. |
| R05 | Preserve different-field simultaneous edits | Worked opposite-side and same-side patches retain each field, including late delivery. | 2 pending; payload contract selected in 1. |
| R06 | Justifiable same-field current value, all earlier values/actors/times | Named comparison metadata, ties, clock-skew examples, losing/equal-value distinct edit history. | 1: Q05–Q07,Q19; 2 pending. |
| R07 | Duplicate webhook | Replay/delivery of same source ID changes current/history only once. | 2–4 design pending; 8 optional test. |
| R08 | Out-of-order webhook | Newer then older yields justified current fields while keeping each distinct edit. | 2,4 pending; 8 optional test. |
| R09 | Stop loops without hiding real changes | Echo correlation stops repeated writes; real integration-account changes and conflict repairs still propagate. | 1: provenance gate; 2,4,5 pending. |
| R10 | Choose patch vs full-record webhook; sample fields included | Check omitted/null/clear/equal-value semantics and safe full-record alternative. | 1: covered; 4 handling details pending. |
| R11 | Portal sample has no time, includes actor | Server authenticates actor and assigns commit time/revision; body cannot forge actor/order. | 1: covered; 2 rules pending. |
| R12 | Salesforce calls our HTTPS endpoint; worker calls record API | Diagram public network boundary, callers, roles and durable boundary; choose one/two services with reason. | 1: one application proposed; 3 diagram/interfaces pending. |
| R13 | Check and apply webhook before storing | Specify validation, atomic application/history/dedup transaction, and successful acknowledgment only after commit; explain interpretation of brief wording. | 3,4 pending; no durable raw-only success assumed. |
| R14 | <=100 outbound requests/s, retries count; throttle keeps work | Shared budget includes reads/recovery/repair, no burst above cap, Retry-After/backoff; demonstrate retries use tokens. | 1: 90/s operating assumption; 5,6 pending; 8 optional test. |
| R15 | Six-hour Salesforce outage, missing webhooks | Trace durable state and restart; replay complete history with committed checkpoint and gap detection. Snapshot-only alternative explicitly insufficient. | 1: replay gate and 72 h assumption covered; 3,6 pending. |
| R16 | Pending calls survive own process restart | Crash boundaries and transactional durable jobs; restart while fake CRM down. | 3,5 design pending; 8 optional process test. |
| R17 | Peak and quiet rates; catch-up minutes with division | Both directions, bursts, amplification, page counts, repairs, ongoing traffic and storage; distinguish estimate from guarantee. | 1: 20/s peak, 1/s quiet each; 432,000 each/6 h, conditional 120 min outbound example; 6 full accounting pending. |
| R18 | Explain build order; components solve named subproblems | Review plan sequence and final component rationale. | Plan present; 3,7 final treatment pending. |
| R19 | Data stored | Cover values, identity mapping, field metadata, audit, dedup, jobs, checkpoints, constraints and transactions. | 3 pending. |
| R20 | Inbound and outbound paths | Pseudocode/state transitions, retry, dependencies, permanent failure, conditional-write races. | 4,5 pending. |
| R21 | Exactly three operational numbers and escalation values | Select three primary metrics, thresholds, reasons in final design. | 7 pending; no final metrics selected in 1. |
| R22 | All 12 requested design sections, buildable for teammate | Final design review against outline below and four required scenarios. | 7 pending. |
| R23 | Full AI chats, unedited, every prompt/reply | Authentic export of all task chats; inspect coverage without trimming or fabricating. | Pending authentic export; 7 tracks submission gap. |
| R24 | Repository with every commit, no squash | Inspect log and retain history; meaningful decision messages if commits authorized. | Existing `4aa91fb Created Implementation Plan` observed; no history changed or new commit authorized. Final packaging in 7. |
| R25 | Open questions plus 10-minute walkthrough | Present assumptions, callers diagram, rules, arithmetic within 10 min. | Questions in 1; walkthrough in 7 pending. |
| R26 | Optional code: TypeScript, Node >=20, fake CRM, Node alone | Fresh checkout: npm install + one demo command, no required external service; four scenarios, durable restart, enforced configured cap. | 8 only; not started and not required submission code. |
| R27 | Weekend exclusions and no production service claim | Review scope for deletes/UI/login/second region/real load test; distinguish demo persistence from PostgreSQL. | 1: covered; 7/8 review pending. |
| R28 | Existing data/bootstrap not specified | Explicit capture boundary, consistent snapshot/replay overlap, mapping and historical coverage decision. | 1: Q16 covered; detailed bootstrap execution excluded, production prerequisite remains open. |
| R29 | Submit via calendar thread at stated time | Confirm revised deadline and authorized delivery channel; no external message sent. | Q20: stated deadline is past; submission timing unresolved. |

## Final design outline coverage for Phase 7

| Brief section | Planned evidence |
|---|---|
| 1. Assumptions and questions | assumptions.md |
| 2. Systems, callers, boundary, interfaces | architecture.md |
| 3. Subproblems and build order | IMPLEMENTATION_PLAN.md and architecture.md |
| 4. Stored data | data-model.md |
| 5. Inbound path | inbound-flow.md |
| 6. Outbound path | outbound-flow.md |
| 7. Concurrent record changes | conflict-rules.md |
| 8. Duplicate and older events | conflict-rules.md and inbound-flow.md |
| 9. Loop prevention | inbound-flow.md and outbound-flow.md |
| 10. Six-hour outage and arithmetic | recovery-capacity.md |
| 11. Three metrics and escalation | design.md, Phase 7 |
| 12. Weekend exclusions | assumptions.md and design.md |

At Phase 1 completion only the plan, assumptions, and this checklist existed. Phase 1 acceptance is satisfied by documenting feasibility and unknowns, not by passing later-phase scenarios. Upstream replay, idempotent creation, conditional updates, source ordering/provenance, and latency/rate bounds remain unverified production prerequisites.

## Phase 2 validation update — 24 November 2026

`conflict-rules.md` now provides design evidence for R05–R11: cases A–N cover independent fields, opposite-side winners, ties, older events, duplicate versus equal-value edits, portal concurrency, skew/rollback, races, ambiguous responses, echoes, and clears. Trusted actor/time and explicit per-source candidates are defined. R03 has ambiguous-create design evidence (case L); R14 has race request amplification accounting; R15 gains the requirement for complete history through the remote conditional-write revision. These are manually traced design cases, not executed integration tests. Detailed transaction/flow/rate/recovery checks in Phases 3–6 remain pending. Assumptions now include Q21; no other planned artifact is claimed to exist. Phase 2 is complete under the documented upstream contracts and provisional business policy.

## Phase 3 validation update — 24 November 2026

`data-model.md` supplies R19's schema/constraints/transaction design, R02's indexed lookup and candidate/history growth inputs, R03's identity and parent-first creation design, and R16's process-crash boundaries. `architecture.md` supplies R12's callers/network/role diagram, R18's component rationale/build order, and R13's apply-and-commit-before-success interpretation. Replay manifests and contiguous completion rules add R15 persistence evidence; generation checks, mutation identity, and lease ambiguity preserve Phase 2 semantics across network calls. R11 authoritative time is clarified as the serialized database decision time. These are design reviews, not executed database or Salesforce tests. R04/R14/R17 and detailed flows remain for Phases 4–6; full storage sizing is still pending. Phase 3 acceptance is met under the documented upstream contracts; no optional code or new commit was created.

## Phase 5 validation update — 24 November 2026

`outbound-flow.md` supplies R03/R16/R20 outbound design evidence: atomic portal history/values/jobs, parent-first creation, individual field outcomes, state machine, process-restart boundaries, and safe ambiguous create/update resolution under stated adapter contracts. R09 covers verified per-field echoes, mixed events, real integration-account edits, and necessary conflict repair. R14 specifies actual-start pacing shared by all attempts/roles, rolling-second arithmetic, cooldown/backoff, hidden-retry prohibition, and fenced failover. R04 has a conditional 25-second outbound allocation, dependent on bounded history coverage and call paths; feasibility and full request amplification remain Phase 6. Existing Phase 4 inbound evidence is in `inbound-flow.md`; this update does not replace its scenario traces or claim runtime tests. Phase 5 acceptance is met as design under explicit upstream/egress assumptions. R17 recovery accounting, R21 metrics, R22–R25 submission validation, and optional R26 remain for their assigned phases. Chat-history authenticity/completeness is not assessed in Phase 5.
