# Start phase 1

> start phase 1

<details><summary>4 previous messages</summary>

> I’ll read the plan and original requirements, check Phase 1’s dependencies and existing artifacts, then carry out its scope and record the validation results.
>
> <details><summary>Ran 2 commands</summary>
>
> - Ran `Get-Content IMPLEMENTATION_PLAN.md; Get-Content Round1-Tequity.html; git status --short; Get-ChildItem -Name`
> - Ran `Get-Content AGENTS.md; git log -5 --oneline; $brief = Get-Content Round1-Tequity.html -Raw; $main = [regex]::Match($brief, '(?s)<article.*?</article>').Value; [System.Net.WebUtility]::HtmlDecode([regex]::Replace($main, '<[^>]+>', ' '))`
>
> </details>
>
> Phase 1 has no dependencies, and there are no existing design artifacts to reuse. I’ll create the assumptions register and requirements checklist. The main feasibility gap is recovering every edit from missing webhooks: that requires replayable history beyond what the brief supplies, so I’ll mark it as an upstream assumption.
>
> <details><summary>Created 2 files, edited a file, ran a command</summary>
>
> - Wrote `./IMPLEMENTATION_PLAN.md` (+2 -1)
> - Wrote `./docs/assumptions.md` (+81 -0)
> - Wrote `./docs/requirements-checklist.md` (+54 -0)
> - Ran `git diff --check; git diff --stat; git status --short; $a = Get-Content docs/assumptions.md -Raw; $c = Get-Content docs/requirements-checklist.md -Raw; 'Question rows: ' + ([regex]::Matches($a, '\| Q\d{2} \|')).Count; 'Requirement rows: ' + ([regex]::Matches($c, '\| R\d{2} \|')).Count; 'Outline rows: ' + ([regex]::Matches($c, '\| \d+\. ')).Count`
>
> </details>
</details>

Phase 1 is complete.

- Created [assumptions.md](/D:/Downloads/Tequity_Assignment/docs/assumptions.md) with workload estimates, payload rules, feasibility limits, and 20 open questions.
- Created [requirements-checklist.md](/D:/Downloads/Tequity_Assignment/docs/requirements-checklist.md), covering 29 requirements and all 12 design sections.
- Updated [IMPLEMENTATION_PLAN.md](/D:/Downloads/Tequity_Assignment/IMPLEMENTATION_PLAN.md) with completion status and validation.

Replay history, safe create retries, and conditional updates remain unverified upstream assumptions. Whitespace checks passed. No code or commits were created; Phase 2 remains unstarted.

> explain me what all was performed in this phase.

Phase 1 established **the assumptions the design will rely on and where the requirements may be infeasible**. It was design groundwork; no application code was written.

Here’s what was done:

1. **Reviewed the assignment and existing repository.** Read the original brief, implementation plan, and repository instructions. Checked the existing commit history and confirmed there were no earlier design artifacts to reuse.

2. **Set provisional traffic assumptions.** Assumed each direction produces **20 edits/second at peak** and **1 edit/second during quiet hours**, with short bursts. These are planning numbers, not measured capacity.

3. **Explained when 30-second synchronization is feasible.** Defined assumed bounds for webhook delivery, queueing, processing, API response, and visibility. Documented that overload, throttling, or delayed webhooks can invalidate that target.

4. **Chose changed-field payloads.** An omitted field means “leave it alone”; an explicit `null` means “clear it,” where allowed. Explained why treating a full-record snapshot as a new edit to every field could overwrite unrelated changes.

5. **Defined trusted identity and timing sources.** Portal actors come from authenticated sessions, and timestamps/revisions come from the server. The submitted `editedBy` value is not trusted. Also recorded assumptions for record IDs, owners, and customer-before-contact dependencies.

6. **Identified three major upstream requirements.**
   - Replayable history to recover every missed edit, including intermediate values and actors.
   - Idempotent creation to avoid duplicate records when a successful response is lost.
   - Conditional updates to protect against concurrent remote changes.

   These are **unverified assumptions**, not confirmed Salesforce capabilities. In particular, fetching current records after an outage cannot reconstruct overwritten history.

7. **Started the outage arithmetic.** At 20 edits/second, six hours creates **432,000 edits in each direction**. The preliminary outbound drain estimate is **120 minutes**, subject to the fuller recovery accounting in Phase 6. A provisional **72-hour replay retention** was chosen.

8. **Recorded scope decisions and open questions.** Documented 20 questions, each with a provisional answer and what changes if that answer is wrong. These include existing-data bootstrap, history retention, schema details, and missing authentic chat exports.

The results are in [assumptions.md](/D:/Downloads/Tequity_Assignment/docs/assumptions.md) and [requirements-checklist.md](/D:/Downloads/Tequity_Assignment/docs/requirements-checklist.md). The checklist maps **29 requirements** to future validation and covers all **12 requested design sections**.

Phase 1 was marked complete because its assumptions, questions, and feasibility gaps were documented—not because the complete synchronization design has been proven. Conflict rules and worked examples are the next phase.

> proceed with phase 2

<details><summary>6 previous messages</summary>

> I’ll review Phase 1’s assumptions, then define the conflict and history rules with worked examples. I’ll focus on concurrent edits, late events, and remote writes that race with our decisions.
>
> <details><summary>Ran 2 commands</summary>
>
> - Ran `Get-Content IMPLEMENTATION_PLAN.md; Get-Content docs/assumptions.md; Get-Content docs/requirements-checklist.md; git status --short`
> - Ran `$brief = Get-Content Round1-Tequity.html -Raw; $article = [regex]::Match($brief, '(?s)<article.*?</article>').Value; [System.Net.WebUtility]::HtmlDecode([regex]::Replace($article, '<[^>]+>', ' '))`
>
> </details>
>
> I’ll keep the latest edit from each source for each field, then compare those candidates by source timestamp with a fixed tie-breaker. This lets source revisions handle late delivery and clock rollback consistently. I’ll also distinguish echoes from repair writes: suppressing every write triggered by an inbound event could leave the two systems with different values.
>
> <details><summary>Created a file</summary>
>
> - Wrote `./docs/conflict-rules.md` (+90 -0)
>
> </details>
>
> The worked cases are drafted. Two details matter for the later design: it must retain both source candidates per field, and a remote revision read alone does not reveal missed edits. I’ve made complete history coverage through that revision an explicit prerequisite for safe reconciliation, and noted that the extra reads and replay calls may increase the Phase 1 capacity estimate.
>
> <details><summary>Edited 3 files, ran a command</summary>
>
> - Wrote `./IMPLEMENTATION_PLAN.md` (+2 -1)
> - Wrote `./docs/assumptions.md` (+7 -0)
> - Wrote `./docs/requirements-checklist.md` (+5 -1)
> - Ran `git diff --check; git status --short; $rules = Get-Content docs/conflict-rules.md -Raw; 'Worked case rows: ' + ([regex]::Matches($rules, '(?m)^\| [A-N]:')).Count; 'Conflict document lines: ' + (Get-Content docs/conflict-rules.md).Count`
>
> </details>
</details>

Phase 2 is complete. Created [conflict-rules.md](/D:/Downloads/Tequity_Assignment/docs/conflict-rules.md) covering:

- Deterministic field conflict rules and trusted portal actor/time.
- History retention for losing, late, and equal-value edits.
- Echo suppression, necessary repairs, and concurrent-write safety.
- 14 worked cases, including clock skew and lost API responses.

Manually reviewed the cases and passed whitespace checks. Updated the assumptions, checklist, and phase log.

Salesforce capabilities remain unverified assumptions. No code or commits were created; Phase 3 remains unstarted.

> give me in detail what was done in phase 2

Phase 2 defined **how to choose current field values without losing any distinct edit**, and how both systems can converge when edits, deliveries, and API calls overlap.

The main artifact is [conflict-rules.md](/D:/Downloads/Tequity_Assignment/docs/conflict-rules.md). It contains the rules, their limitations, and 14 worked examples.

### 1. Defined what counts as an edit

A logical edit now has a clear identity and ordering metadata:

| Metadata | Purpose |
|---|---|
| `editId` | Identifies one distinct edit |
| `origin` | Says whether it originated in the portal or Salesforce |
| `sourceRevision` | Orders committed edits within that source record |
| `occurredAt` | Supplies the source-assigned timestamp |
| `actorId` | Identifies who made the edit |
| Changed-field patch | Records exactly which fields were set or cleared |

Source revisions are compared **within their own source**. Portal revision 15 and Salesforce revision 20 do not establish which edit happened later across systems.

For portal edits, the actor comes from the authenticated session. The server assigns the timestamp and revision. The request body’s `editedBy` and browser clock cannot establish identity or order.

A retried portal request uses the same request key so that it does not accidentally create another logical edit.

### 2. Chose the field conflict rule

The design keeps **two candidates for each field**:

- The latest portal-origin edit to that field.
- The latest Salesforce-origin edit to that field.

It then makes the decision in two steps:

1. **Within each source:** the highest source revision becomes that source’s candidate.
2. **Between the two candidates:** the later `occurredAt` wins. If timestamps are exactly equal, the portal wins.

The portal preference on an exact tie is an explicit business choice that still needs client agreement.

For example:

| Edit | Field | Value | Time |
|---|---|---|---|
| Portal P1 | Phone | B | 11:30:05 |
| Salesforce S1 | Phone | C | 11:30:04 |

The current phone becomes **B**, while both edits remain in history.

For different fields, there is no need to choose one whole-record winner. A portal city correction and a Salesforce phone correction both survive.

### 3. Accounted for clock skew and rollback

The rules explicitly avoid claiming that timestamps reveal the actual last human action.

Two clocks can disagree. A physically later Salesforce edit might carry an earlier timestamp and lose under the chosen policy. That limitation is documented.

Clock rollback also explains why the design retains both source candidates. Consider:

- Salesforce revision 1: phone B, timestamp `:06`.
- Salesforce revision 2: phone C, timestamp `:04`, after its clock moved backward.
- Portal revision 1: phone D, timestamp `:05`.

Salesforce’s candidate is **C**, because revision 2 supersedes revision 1 within Salesforce. Comparing C against the portal candidate makes **D** the overall winner.

Keeping only the largest timestamp from all historical edits would incorrectly retain B.

### 4. Defined what history must preserve

Every distinct edit remains recorded, including an edit that:

- Loses a conflict.
- Arrives late.
- Is later superseded.
- Sets a field to the value it already had.
- Never gets applied remotely because it is already obsolete.

The design distinguishes these cases:

| Situation | History result |
|---|---|
| The same webhook is delivered twice | One logical edit |
| A new edit ID sets the same value again | Two logical edits |
| Our API update returns through a webhook | Application evidence linked to the original edit |
| An API request fails | Attempt evidence, without inventing a human edit |

History retains the original actor, time, revision, and changed values. It also records why a particular edit supplied the current value.

An edit that won initially can later become superseded; “won when received” is not treated as a permanent status.

### 5. Specified concurrent portal-save behavior

The selected policy accepts valid changed-field patches and serializes portal saves.

If two portal users change different fields, both changes survive.

If they change the same field, the higher portal revision becomes the portal candidate. Both edits remain in history.

A save response should return:

- The accepted edit ID.
- The authoritative revision and timestamp.
- The resolved current values.
- Synchronization status.

**An accepted save means the edit was durably retained.** It does not necessarily mean its value wins against an existing Salesforce candidate.

The design also requires genuine patches. A stale form must not silently resend unchanged fields and overwrite another person’s changes.

### 6. Explained what Salesforce should receive

The worker should send fields that still need to reach the resolved winning value.

Suppose two portal users quickly save phone B and then phone C:

- If B was already sent, Salesforce may see B followed by C.
- If B became obsolete before dispatch, its field application can be marked superseded, and C is sent.

The earlier edit and its work record remain retained. This does not enable general pending-write coalescing.

This also exposed a scope question: the design preserves complete edit history **on our side**. Making every intermediate edit appear in Salesforce’s audit history would require an additional upstream interface. That question was added as **Q21**.

### 7. Defined how to handle remote-write races

A worker can prepare a write while a Salesforce user is editing the same record.

The design requires a conditional write against an expected remote revision:

1. Establish the remote revision.
2. Incorporate original edit history through that revision.
3. Recompute the field winners.
4. Send only the necessary patch with that revision as a precondition.
5. If the revision changed, fetch the intervening history and reconsider the write.

For example, if the worker expects remote revision 10 but a sales user commits revision 11 first, the write against revision 10 must fail without changing the record.

The worker then incorporates revision 11. It may discover that its original patch should be changed or cancelled.

A significant refinement here was that **reading current values and a revision is insufficient**. The worker also needs complete history through that revision; otherwise, it could overwrite an unseen remote edit without evaluating it.

These capabilities remain upstream assumptions.

### 8. Separated echoes from repairs

An **echo** is evidence that our own propagated edit was applied remotely. It must preserve the original edit’s identity rather than become a new Salesforce-origin human edit with a later timestamp.

A **repair** is a necessary write that restores the resolved winner when the remote value differs.

For example:

- The portal has phone B from a later edit.
- A delayed Salesforce edit sets its copy to C but loses under the conflict rule.
- The portal retains B and records the Salesforce edit.
- A repair sends B back to Salesforce.

Suppressing every outbound action triggered by a webhook would leave the systems disagreeing in this case.

The rules also avoid filtering all changes made by an integration account. A genuine edit by that account must still be processed unless reliable provenance proves it is an echo.

### 9. Covered lost API responses

The document addresses requests that succeed remotely but whose responses never reach us.

For an **update**, the worker retains the mutation identity and resolves the result through correlation, replay, or current revision-associated history. It must not blindly resend an old value over a newer Salesforce edit.

For a **create**, the design relies on the Phase 1 assumption of a unique external ID or durable idempotency mechanism. Retrying must resolve to the same remote record, without resetting fields that someone subsequently edited.

Without those upstream capabilities, duplicate-free ambiguous creates cannot be guaranteed.

### 10. Identified additional request costs

A conflict can require several requests rather than one.

The document gives this example:

| Action | Requests |
|---|---:|
| Conditional write that fails | 1 |
| Read the new remote revision | 1 |
| Fetch one history page | 1 |
| Retry the recomputed write successfully | 1 |
| **Total** | **4** |

All attempts count toward the shared outbound budget.

This means Phase 1’s one-write baseline and 10 requests/second overhead allowance remain provisional. Phase 6 must include the actual read, replay, repair, and retry costs before adopting a recovery-time estimate.

### 11. Worked through 14 scenarios

The examples specify expected current values, retained history, and remote actions for:

1. Different-field edits.
2. A same-field conflict won by Salesforce.
3. A same-field conflict won by the portal.
4. An exact timestamp tie.
5. Out-of-order edits affecting different fields.
6. Duplicate delivery versus a distinct equal-value edit.
7. Concurrent portal saves.
8. Clock skew.
9. Source clock rollback.
10. A remote edit racing with our write.
11. An accepted update with a lost response.
12. An accepted create with a lost response.
13. A delayed echo and a genuine integration-account edit.
14. An explicit clear versus an omitted field.

These were **manually traced design examples**. No executable tests or real Salesforce verification were performed.

### 12. Updated the project records

Alongside the new conflict document, I updated:

- [assumptions.md](/D:/Downloads/Tequity_Assignment/docs/assumptions.md) with the Phase 2 decisions and strengthened upstream requirements.
- [requirements-checklist.md](/D:/Downloads/Tequity_Assignment/docs/requirements-checklist.md) with the design evidence now available.
- [IMPLEMENTATION_PLAN.md](/D:/Downloads/Tequity_Assignment/IMPLEMENTATION_PLAN.md) with Phase 2’s completion status, validation, and unresolved questions.

Phase 2 is complete **under the documented assumptions**. The policy is deterministic, different-field changes survive, losing edits remain auditable, and reconciliation can converge when the required history, conditional-write, and provenance capabilities exist. Those capabilities still need confirmation. No application code or commits were created.

explain to me why would salesforce clock move backward?







1:21 PM





A server clock can move backward when its time is corrected. For example, if it is running two seconds fast, a time-synchronization mechanism could reset it to the correct time. Some systems avoid this by gradually slowing the clock instead.

Another possibility is different servers with slightly different clocks:
- Server A records the first edit at 11:30:06.
- Server B records a later edit at 11:30:04 because its clock is behind.
The timestamps appear to move backward even though neither server changed its clock.

We have not verified that Salesforce timestamps behave this way. The example was a defensive hypothetical, not a claim about Salesforce.

Its purpose was to explain why the design distinguishes:
- Source revision: the authoritative sequence of edits, assuming Salesforce supplies such a revision.
- Timestamp: a reported time, which does not automatically guarantee ordering.
If Salesforce guarantees that the relevant timestamp never decreases across successive record revisions, that particular rollback case would not apply. Clock differences between Salesforce and our portal would still matter when comparing edits across the two systems.

  
1:22 PM

how does it identify if the clock is behind or ahead, there should be some baseline clock or something?







2:40 PM





Yes—to know whether a clock is ahead or behind, you need a reference clock.

Servers commonly synchronize through NTP (Network Time Protocol) with time servers that ultimately trace back to references such as atomic clocks or GPS. They exchange timestamps to estimate:
- The difference between their clocks.
- The network round-trip delay.
Because network delays can vary, that estimate has uncertainty. Repeated measurements help improve it. The server can then adjust its clock gradually or, depending on configuration, move it directly.

There are two separate questions in our design:
Question	How we answer it
Is the server clock accurate relative to UTC?	Time synchronization and clock-offset monitoring
Which edit came later within one source?	A trusted, increasing source revision


Our sync logic does not need to detect clock rollback to use revisions. If revision 2 has an earlier timestamp than revision 1, we still know revision 2 is the later source edit—assuming the source guarantees revision ordering.

Across Salesforce and the portal, however, those independent revisions cannot establish order. Comparing their timestamps relies on clock accuracy and remains uncertain. Our timestamp rule therefore chooses a consistent winner; it cannot prove which person actually acted last.

The Phase 1 assumption of clocks being within ±1 second of UTC is a proposed operating requirement, not something we have measured or confirmed for Salesforce.

What does webhook mean here? and how does the system identify if the same webhook is delivered twice.







3:50 PM





A webhook here is an HTTPS message Salesforce sends to our portal when a record changes.

For example, when a salesperson changes a contact’s phone number, Salesforce sends something like:
{
  "eventId": "evt_1001",
  "recordId": "contact_123",
  "occurredAt": "2026-11-02T11:30:04.000Z",
  "changedBy": "sales.user@client.example",
  "changedFields": {
    "phone": "+919876543210"
  }
}
This tells our portal: “This contact’s phone number changed; update your copy and retain the edit in history.”

Why could the same message arrive twice?

Suppose our portal receives the message and saves the change, but its acknowledgment gets lost over the network. Salesforce cannot tell whether we received it, so it sends the message again. Both deliveries refer to the same original edit.

How do we recognize the duplicate?

We use the stable eventId, together with its tenant/source scope. The database remembers processed event identities.
Incoming delivery	Database check	Result
First evt_1001	ID has not been processed	Apply the change, record its history, and remember the ID
Second evt_1001	ID already exists	Acknowledge it without applying or recording the edit again


Those first-delivery actions must happen in one database transaction. A unique constraint on the scoped event ID also protects against two copies arriving simultaneously.

We compare the content too: if the same event ID arrives with different content, that is an integrity error to investigate.

Crucially, we do not identify duplicates by the phone number or timestamp. A salesperson could deliberately save the same phone number again as a separate edit. With a new event ID, that remains a distinct history entry.

Our design assumes Salesforce’s integration supplies a stable event ID that stays unchanged on retries; that capability still needs confirmation.

# Proceed with next phase

> proceed with the next phase and give me in details what was implemented.

<details><summary>6 previous messages</summary>

> I’ll read the implementation plan and check the existing artifacts to identify the next phase, then implement it and report the changes and validation in detail.
>
> <details><summary>Explored a file, ran 3 commands</summary>
>
> <details><summary>Explored a file</summary>
>
> - Read `./IMPLEMENTATION_PLAN.md`
>
> </details>
>
> - Ran `git status --short; rg --files -g '!node_modules' -g '!.venv' -g '!package-lock.json'`
>
> - Ran `Get-Content docs/conflict-rules.md; Get-Content docs/assumptions.md; Get-Content docs/requirements-checklist.md; Get-Content Round1-Tequity.html`
>
> - Ran `Get-Content docs/assumptions.md; Get-Content docs/requirements-checklist.md; Get-Content AGENTS.md`
>
> </details>
>
> The next phase is Phase 3: durable data and architecture boundaries. Phases 1–2 are complete, and their artifacts are available.
>
> I’ll create `docs/data-model.md` and `docs/architecture.md` covering PostgreSQL tables, transaction boundaries, worker recovery, identity mapping, parent dependencies, and the HTTP/worker interfaces. This phase is design work; executable code remains scheduled for Phase 8.
>
> <details><summary>Created 2 files</summary>
>
> - Wrote `./docs/architecture.md` (+90 -0)
> - Wrote `./docs/data-model.md` (+88 -0)
>
> </details>
>
> The two Phase 3 documents are drafted. They specify atomic saves and webhook application, separate original edits from transport evidence, and preserve unresolved mutations for reconciliation after a lost response.
>
> The design also makes missing parent and owner mappings visible as blocked work. I’m checking consistency with Phase 2’s conflict rules and reviewing the crash boundaries before marking the phase complete.
>
> <details><summary>Edited 4 files, ran a command</summary>
>
> - Wrote `./IMPLEMENTATION_PLAN.md` (+2 -1)
> - Wrote `./docs/assumptions.md` (+2 -0)
> - Wrote `./docs/data-model.md` (+1 -1)
> - Wrote `./docs/requirements-checklist.md` (+4 -0)
> - Ran `git diff --check; git diff --stat; rg -n '^##|Phase 3|\| 3 \|' docs/data-model.md docs/architecture.md IMPLEMENTATION_PLAN.md; git status --short`
>
> </details>
</details>

Completed **Phase 3: durable data and boundaries**. This phase produces the persistence and architecture design; executable implementation remains Phase 8.

Created two documents:

- [data-model.md](./docs/data-model.md)
- [architecture.md](./docs/architecture.md)

The work covers:

1. **Production PostgreSQL data model.** Defined customers, contacts, source-ID mappings, principals, field candidates, resolved values, immutable edit history, webhook receipts, outbound jobs, mutation attempts, dependencies, and replay checkpoints. Specified unique constraints, foreign keys, and indexes.

2. **Atomic saves and webhook application.** Values, history, field decisions, deduplication receipts, and resulting work commit together. Successful responses follow that commit, making retries safe after a lost response.

3. **Concurrency and worker recovery.** Defined per-record locks, worker claims, leases, fencing epochs, and generation checks. An expired lease does not prove a remote request failed: ambiguous mutations remain durable and require reconciliation before conflicting writes continue.

4. **Creation and dependencies.** Defined stable identity mapping, customer-before-contact dispatch, and blocked handling for unresolved customers or owners. Original reference edits remain in history while mappings are resolved.

5. **Replay persistence.** Defined page manifests and contiguous checkpoints. Completed later pages cannot advance recovery past an incomplete earlier page.

6. **Architecture and interfaces.** Added a diagram showing callers, HTTPS boundaries, HTTP and worker roles, and PostgreSQL. Specified interface sketches for portal saves, webhooks, conditional updates, safe creates, and replay.

7. **Scale considerations.** Documented indexed lookups for five million records, candidate-row estimates, history growth, and retention constraints. Full recovery and storage accounting remains Phase 6.

Validation included manually reviewing Phase 2’s cases A–N against the data model and tracing crash/retry boundaries. `git diff --check` passed. No database migration, executable tests, or real Salesforce verification was performed.

Updated the [implementation plan](./IMPLEMENTATION_PLAN.md), assumptions, and requirements checklist. Phase 3 is complete under the documented upstream assumptions; replay, conditional-write, idempotency, and provenance capabilities still require confirmation. No commits were created.

The next phase is **Phase 4: inbound synchronization**.

is SCD type 2 implemented here?







5:31 PM





No—SCD Type 2 is not implemented in the current design. We preserve history using immutable edit records instead.

SCD Type 2 typically creates a new version of the whole record whenever tracked attributes change:
Contact	Phone	Valid from	Valid to	Current
C1	111	10:00	11:00	No
C1	222	11:00	—	Yes


Our design stores:
- Current contact row: the resolved values shown in the portal.
- Immutable edit history: every distinct changed value, actor, time, and source revision—including losing and late edits.
- Field candidates and resolutions: the competing edits and selected winner for each field.
That distinction matters: a losing edit must remain in history even though it never became the current value. Ordinary SCD Type 2 rows alone would not capture that distinction.

You can describe it as: “An operational relational model with field-level audit history, rather than SCD Type 2 record versioning.” Also, this is currently a documented design; executable database code has not been built yet.

  
5:31 PM








what is the difference between the current design and SCD type 2?







5:32 PM





The main difference is what history represents:
- Our design records every edit, including edits that lose a conflict.
- SCD Type 2 records successive versions of a business record, usually with a validity period.
Aspect	Current design	SCD Type 2
Main purpose	Synchronization, conflict resolution, and auditing	Historical reporting and analysis
History unit	An edit and its changed fields	A version of a record
Current values	Customer/contact row plus field-resolution metadata	Row marked current, often using is_current
Historical metadata	Actor, source, time, revision, edit identity	Typically valid_from, valid_to, and a version key
Losing edits	Explicitly retained even when never current	Need additional modeling to retain them as rejected edits
Concurrent edits	Defined field-level conflict rules	SCD2 itself does not define conflict resolution
Duplicate events	Deduplicated using event/edit identities	SCD2 itself does not define event deduplication
Reliable delivery	Durable jobs, retries, receipts, checkpoints	Outside SCD2’s purpose


Consider this example:
1. Phone starts as 100.
2. Portal edits it to 111.
3. Salesforce edits it to 222; our conflict rule selects 222.
4. The portal edit arrives late and never becomes current.
Our history keeps both edits: 111 and 222, their actors and times, and evidence explaining why 222 won.

A typical SCD2 history of resolved state contains: 100 → 222, with validity periods. It would need a separate edit log to preserve the losing 111 change.

The approaches can coexist. We could later generate an SCD2 table from resolved-state changes for reporting, while retaining the current edit history for synchronization correctness. SCD2 would complement this design; it would not replace its conflict and delivery mechanisms.

  
5:32 PM








so which approach will be better? keeping the current design or also adding SCD2?







5:34 PM





Keep the current design for this assignment. Add SCD Type 2 later only if there is a clear historical reporting

# Implement phase 4

> proceed with phase 4 and explain in detail what what was implemented.

<details><summary>6 previous messages</summary>

> I’ll read Phase 4 and check its dependencies and existing artifacts, then implement its scope, validate the results, and update the plan. I’ll also explain what was implemented in detail.
>
> <details><summary>Ran 4 commands</summary>
>
> - Ran `Get-Content IMPLEMENTATION_PLAN.md; Get-ChildItem -Force; git status --short`
> - Ran `Get-Content docs/assumptions.md; Get-Content docs/conflict-rules.md; Get-Content docs/data-model.md; Get-Content docs/architecture.md; Get-Content docs/requirements-checklist.md; Get-Content AGENTS.md; Get-Content Round1-Tequity.html`
> - Ran `Get-Content docs/conflict-rules.md; Get-Content docs/data-model.md; Get-Content docs/architecture.md`
> - Ran `Get-Content docs/data-model.md -TotalCount 65`
>
> </details>
>
> Phases 1–3 are complete, and their design artifacts provide the dependencies Phase 4 needs. Phase 4 specifies the inbound path; executable code belongs to Phase 8.
>
> I’m creating `docs/inbound-flow.md` with the authentication and validation contract, atomic apply-before-acknowledge flow, error responses, dependency handling, and worked scenarios. It will distinguish verified echoes from genuine edits and explain when a losing remote edit still needs a repair job.
>
> <details><summary>Created a file</summary>
>
> - Wrote `./docs/inbound-flow.md` (+143 -0)
>
> </details>
>
> The inbound specification is drafted. It covers commit-before-acknowledge behavior, duplicate retries, per-field handling of late edits, unresolved parent/owner references, and webhook/replay overlap.
>
> I’m checking it against the earlier conflict and persistence rules, especially that an old echo cannot roll back newer state and that a losing remote edit creates repair intent when remote agreement is unproven. The document labels the latency budget as conditional; no running endpoint or Salesforce capability has been verified.
>
> <details><summary>Edited a file, ran a command</summary>
>
> - Wrote `./IMPLEMENTATION_PLAN.md` (+2 -1)
> - Ran `git diff --check; git diff --stat; rg -n 'Phase 4|\| 4 \||## |Scenario|Total|Validation performed' IMPLEMENTATION_PLAN.md docs/inbound-flow.md; git status --short`
>
> </details>
</details>

Phase 4 is complete as **design work**, as specified in the plan. It produces an implementation-ready inbound synchronization specification; executable code remains scheduled for Phase 8.

Created [docs/inbound-flow.md](./docs/inbound-flow.md) and updated the status and execution log in [IMPLEMENTATION_PLAN.md](./IMPLEMENTATION_PLAN.md).

The specification covers these behaviors:

1. **Webhook authentication and validation.** Defined the HTTPS endpoint, a provisional HMAC signature contract, a five-minute delivery-signature window, and a 64 KiB body limit. Validation checks tenant/account authority, schema version, object and field allowlists, revisions, timestamps, types, and nullability. The signature timestamp is separate from the edit timestamp, so an old edit can still arrive in a fresh authenticated delivery.

2. **Separate delivery identity from edit identity.** Event IDs detect repeated deliveries; logical edit IDs preserve distinct edits. Immutable-content checks reject an existing identity reused with different content. Two separate edits setting the same value remain two audit facts.

3. **Atomic application before acknowledgment.** Added pseudocode for one database transaction that records history, evaluates field candidates, updates current values, preserves dependency or repair work, and inserts the accepted receipt. Success follows commit. A crash before commit rolls everything back; a crash after commit but before the response is handled safely by deduplication on retry.

4. **Per-field handling of late events.** An older event remains in history without replacing a newer candidate for the same field. Its other fields can still apply. For example, if a newer phone update arrives before an older phone-and-city update, the newer phone survives while the older city change can still be accepted.

5. **Unknown records and dependencies.** Updates arriving before creates retain their history and partial state in placeholders. Missing customers or owners produce explicit blocked status and durable dependency work. Later hydration rechecks the latest field decisions, preventing an old dependency from overriding a newer reassignment.

6. **Echo suppression and necessary repairs.** Verified echoes record application evidence without creating another human edit or sending the same change back. Genuine integration-account edits are processed normally. When a remote edit loses to the portal’s resolved value, the transaction preserves reconciliation intent so Salesforce can eventually converge. Mixed echo-and-human events are classified per field and accepted atomically.

7. **Errors, replay overlap, and full-record payloads.** Added an HTTP response matrix with retry and remediation rules. Invalid events receive no accepted receipt and cannot silently advance replay checkpoints. Replay reuses the webhook application routine. Full snapshots require authoritative change metadata; snapshot differences alone cannot establish edit history.

8. **Scenarios and latency assumptions.** Documented 16 scenario outcomes covering duplicates, late events, crashes, dependencies, echoes, invalid payloads, and replay. The conditional healthy inbound budget is **20 seconds**, leaving 10 seconds against the requirement.

Validation consisted of manual scenario tracing and consistency review against Phases 2–3; `git diff --check` passed. No running endpoint, executable tests, measured latency, or real Salesforce capability verification is claimed. Authentication, replay, provenance, and schema contracts remain explicit assumptions requiring confirmation.

Existing work and git history were preserved. No commit was created, and Phases 5–8 remain unstarted.

# Execute phase 5

> proceed with phase 5

<details><summary>6 previous messages</summary>

> I’ll read Phase 5 and check its dependencies and existing artifacts before making changes.
>
> <details><summary>Explored 6 files, ran 2 commands</summary>
>
> <details><summary>Explored a file</summary>
>
> - Read `./IMPLEMENTATION_PLAN.md`
>
> </details>
>
> - Ran `git status --short; rg --files -g AGENTS.md -g package.json -g '*phase*' -g '*Phase*'`
>
> - Ran `Get-Content docs/assumptions.md, docs/conflict-rules.md, docs/data-model.md, docs/architecture.md, docs/inbound-flow.md`
>
> <details><summary>Explored 6 files</summary>
>
> - Read `./Round1-Tequity.html`
> - Read `./docs/data-model.md`
> - Read `./docs/conflict-rules.md`
> - Read `./docs/inbound-flow.md`
> - Read `./docs/conflict-rules.md`
> - Read `./docs/architecture.md`
>
> </details>
>
> </details>
>
> Phases 1–4 are complete and provide the required dependencies. Phase 5 is design work: I’ll create `docs/outbound-flow.md` covering durable jobs, conditional writes, ambiguous responses, retries, and a shared rate cap. I’ll keep Salesforce capabilities explicitly provisional; recovery scheduling and capacity calculations remain in Phase 6.
>
> <details><summary>Created a file</summary>
>
> - Wrote `./docs/outbound-flow.md` (+148 -0)
>
> </details>
>
> The outbound draft is written. The rate-cap review exposed an important constraint: database leases alone cannot enforce the cap if a paused dispatcher resumes after takeover. The design now requires one paced egress dispatcher and confirmed fencing before failover. I’m checking this against the earlier job, echo, and crash-recovery rules before marking Phase 5 complete.
>
> <details><summary>Edited 4 files, ran 3 commands</summary>
>
> - Ran `Get-Content docs/requirements-checklist.md; git diff --check; git status --short`
> - Ran `git status --short; git log -3 --oneline; Get-Content IMPLEMENTATION_PLAN.md -Tail 14`
> - Wrote `./IMPLEMENTATION_PLAN.md` (+2 -1)
> - Wrote `./docs/assumptions.md` (+6 -0)
> - Wrote `./docs/data-model.md` (+1 -1)
> - Wrote `./docs/requirements-checklist.md` (+4 -0)
> - Ran `git diff --check; git diff --stat; rg -n 'Phase 5|\| 5 \|' IMPLEMENTATION_PLAN.md docs/assumptions.md docs/requirements-checklist.md; Get-Item docs/outbound-flow.md`
>
> </details>
</details>

Phase 5 is complete.

Created [outbound-flow.md](./docs/outbound-flow.md) covering durable jobs, conditional writes, retries, ambiguous responses, shared rate enforcement, and echo prevention.

Updated the [plan and execution log](./IMPLEMENTATION_PLAN.md), assumptions, data model, and validation checklist.

Manual design checks and whitespace validation passed. Salesforce capabilities remain explicit assumptions; no code or commits were created. Phase 6 remains unstarted.

> explain in detail what was implemented

Phase 5 completed the **outbound synchronization design**: the rules for taking a portal edit, preserving it durably, sending the appropriate changes to Salesforce, and recovering safely when requests fail.

**No application code was implemented.** Phases 1–6 are design work under the plan; the executable TypeScript demo belongs to Phase 8. The main artifact is [docs/outbound-flow.md](./docs/outbound-flow.md).

### 1. Atomic portal saves

The design specifies that a portal save writes all related state in one PostgreSQL transaction:

- The submitted field changes.
- The authenticated actor, server-assigned timestamp, and portal revision.
- Immutable edit history.
- Updated field candidates and resolved current values.
- An outbound job describing the work to propagate.
- The response associated with the caller’s request key.

The portal returns success only after this transaction commits.

This prevents a failure where the portal accepts an edit but crashes before recording the outbound work. After a restart, the worker finds the committed job in the database.

A repeated request with the same request key returns the stored result without creating another edit. Reusing that key with different content is rejected.

### 2. Field patches and obsolete work

The worker sends explicit field patches. An omitted field remains untouched; an explicit `null` clears a nullable field.

Before dispatch, it checks whether each job’s intended edit still wins under the existing conflict rules. An obsolete field is marked superseded, with the replacement edit identified, while its original history and job remain retained.

For example:

1. A portal user changes city to Noida.
2. Another portal user changes city to Pune before the worker sends the first edit.
3. Both edits remain in history.
4. The first unsent city change can be marked superseded.
5. The worker propagates Pune.

If the earlier job also contains a phone change that still wins, that field remains eligible for propagation. Jobs are not combined or deleted to achieve this.

### 3. Per-record ordering and dependencies

The design permits work on different records concurrently, while allowing only **one unresolved outbound mutation per record**.

This prevents a newer write from bypassing an older request whose outcome is unknown.

Dependency handling is also explicit:

- A customer must have a confirmed Salesforce mapping before its dependent contact is created.
- An uncertain customer creation blocks that contact.
- Owner and customer references use stable mappings.
- Missing mappings create visible blocked work.
- Resolving a dependency wakes affected jobs, which recheck the latest winning values before proceeding.

The worker never substitutes an arbitrary owner or customer to get a request through.

### 4. Conditional updates with complete history coverage

Before updating an existing Salesforce record, the worker must have:

1. A remote record revision, `R`.
2. The observed values at that revision.
3. Complete attributable change history through `R`.

The third requirement matters because a snapshot can show the latest value while hiding intermediate edits and their actors.

The worker incorporates that history using the existing inbound rules, recomputes the winners, and prepares a patch conditional on revision `R`.

If Salesforce changes after the observation, the conditional write must fail without changing anything. The worker then obtains the intervening history and recomputes.

For example, if Salesforce changes the phone while the portal is changing the city, the city patch must preserve the new phone. If both change the phone, the conflict policy determines which value should remain current, while both edits stay in history.

These conditional-write and history interfaces remain **required upstream assumptions**, not verified Salesforce capabilities.

### 5. Durable mutations and acknowledgment

Before making a network call, the worker records an immutable mutation containing:

- Its mutation ID and associated job.
- The exact patch.
- Original edit identities and provenance for each field.
- The expected Salesforce revision.
- The local resolution generation used to prepare it.

Each network attempt has a separate attempt ID.

This separates the logical mutation from its retries. An identical safe retry retains the mutation ID; a rebuilt patch or changed precondition receives a new mutation ID after the previous request has been resolved.

A successful response records application evidence. It cannot overwrite newer portal decisions. If another portal edit arrived while the request was in flight, that edit retains its follow-up work.

### 6. Job states and crash recovery

The document defines a job state machine using:

| State | Meaning |
|---|---|
| `READY` | Eligible for processing. |
| `LEASED` | Claimed by a worker. |
| `BLOCKED` | Waiting for a dependency, capability, or intervention. |
| `RETRY` | Waiting until a persisted retry deadline. |
| `UNCERTAIN` | A request may have taken effect; reconciliation is required. |
| `SUCCEEDED` | Relevant fields were satisfied at identified evidence. |
| `SUPERSEDED` | The unsent work became obsolete. |
| `FAILED` | A definitive permanent rejection requires remediation. |

The design traces crashes before commit, after commit, after mutation preparation, after sending, and after remote acceptance.

A worker lease expiring does **not** prove that Salesforce rejected or never received a request. Prepared requests with missing outcomes must be treated as potentially sent.

### 7. Strict shared rate enforcement

Every outbound request shares one budget, including writes, retries, reads, history pages, lookups, and repairs.

The configured cap is at most 100 requests per second, with a proposed operating rate of 90.

The design uses one egress dispatcher that spaces actual request starts. At 90 requests per second, the minimum spacing is **11,112 microseconds**. It does not accumulate unused capacity or send a burst after a pause.

Automatic HTTP retries and redirects are disabled because they would otherwise create calls outside explicit accounting.

A key refinement concerns failover: a database lease alone cannot enforce the cap if an old dispatcher pauses, loses its lease, and later resumes. Replacement dispatch therefore requires confirmed termination or fencing of the old sender, followed by a one-second pause. If ownership cannot be established safely, dispatch pauses while portal work remains durable.

The defined cap applies to client request initiation. Network delays can bunch arrivals at Salesforce, so an arrival-based interpretation remains an open contract question.

### 8. Retry, throttling, and ambiguous responses

The design includes:

- A provisional five-second total request timeout.
- Exponential backoff with full jitter, capped at 60 seconds.
- Persisted retry deadlines.
- Account-wide cooldowns that honor `Retry-After`.
- Reduced sending rates after throttling.
- Retention of pending work without automatic deletion after a retry count.

It distinguishes a **known no-effect failure** from an **unknown outcome**.

For an accepted update whose response is lost, the worker resolves the original mutation through authoritative evidence, safe idempotency, or complete history. It does not blindly resend an old patch over newer edits.

For an accepted create whose response is lost, retries must resolve to the same record using a stable external UUID or idempotency mechanism. The create-or-resolve operation must return the existing identity without resetting its newer fields. A generic destructive upsert is insufficient.

### 9. Echo prevention without hiding genuine changes

An inbound field is treated as an echo only when its provenance matches a stored outbound mutation.

A verified echo records application evidence without creating another human edit or advancing Salesforce’s original field candidate.

The design also handles:

- Mixed events containing propagated fields and genuine new edits.
- Real edits made by an integration account.
- Delayed echoes arriving after newer changes.
- Genuine remote edits that lose arbitration and require a repair.

This allows necessary conflict repairs while stopping propagation from generating an endless sequence of new edits.

### 10. Supporting updates and validation

The phase also updated:

- [IMPLEMENTATION_PLAN.md](./IMPLEMENTATION_PLAN.md): Phase 5 status, execution log, validation, and unresolved questions.
- [docs/assumptions.md](./docs/assumptions.md): egress fencing, safe create retries, and ambiguous-update requirements.
- [docs/data-model.md](./docs/data-model.md): persisted rate and cooldown coordination.
- [docs/requirements-checklist.md](./docs/requirements-checklist.md): Phase 5 design evidence.

Validation consisted of manual scenario tracing against the earlier conflict, inbound, and persistence rules, plus whitespace checks. **No executable tests, production benchmarks, or live Salesforce verification were performed.**

Phase 6 still needs to calculate full request amplification, recovery scheduling, storage, and backlog drain time. The earlier 120-minute recovery estimate remains provisional.

explain me this







6:35 PM





“Recovering safely when requests fail” means keeping the pending work and retrying or checking its outcome without creating duplicates, losing edits, or overwriting newer changes.

A failed request can mean two different things:

1. Salesforce never received it.
For example, the connection failed before the request was sent. The job stays in our database, and the worker retries later, respecting the rate limit.

2. Salesforce processed it, but we never received the response.
For example:
1. We ask Salesforce to create customer A.
2. Salesforce creates the customer.
3. The connection breaks before its success response reaches us.
Our worker sees a timeout, but a timeout does not prove the operation failed. Simply creating the customer again could produce a duplicate.

The design therefore gives the creation a stable identity. The worker uses that identity to find or safely resolve the original result before continuing. This depends on the assumed upstream support for unique creation or idempotency.

Updates need similar care. If our phone update succeeds, its response is lost, and a Salesforce user then changes the phone again, blindly retrying our old update could erase their newer change. The worker checks revision/history evidence and recomputes what should be sent.

The pending job is stored durably, so restarting our process does not erase it. Safe recovery means continuing from recorded evidence rather than guessing whether a request succeeded.

# Implement phase 6

> implement phase 6 and explain in detail what was done.

<details><summary>6 previous messages</summary>

> I’ll read Phase 6 and check its dependencies and existing artifacts before making changes, then implement it and record the validation results in the plan.
>
> <details><summary>Explored 2 files, ran 4 commands</summary>
>
> <details><summary>Explored a file</summary>
>
> - Read `./IMPLEMENTATION_PLAN.md`
>
> </details>
>
> - Ran `git status --short; rg --files -g '!node_modules' -g '!.venv' -g '!package-lock.json'`
>
> - Ran `Get-Content docs/assumptions.md; Get-Content docs/outbound-flow.md`
>
> - Ran `Get-Content docs/data-model.md; Get-Content docs/inbound-flow.md; Get-Content docs/conflict-rules.md; Get-Content docs/architecture.md; Get-Content docs/requirements-checklist.md; Get-Content Round1-Tequity.html`
>
> <details><summary>Explored a file</summary>
>
> - Read `./docs/outbound-flow.md`
>
> </details>
>
> - Ran `rg -n 'replay|checkpoint|coverage|bytes|storage|200|latency|partition|manifest' docs/data-model.md docs/inbound-flow.md docs/architecture.md docs/conflict-rules.md`
>
> </details>
>
> Phases 1–5 are complete, so Phase 6’s dependencies are available. The main capacity finding is that the earlier 120-minute estimate omits requests needed to read remote revisions and recover history before safe writes. I’ll document recovery barriers, restart behavior, and a full request and storage budget in `docs/recovery-capacity.md`, then update the plan. This phase is design work; executable code remains in Phase 8.
>
> <details><summary>Ran 2 commands</summary>
>
> - Ran `Get-Content docs/outbound-flow.md | Select-Object -Last 32; Get-Content docs/data-model.md | Select-Object -First 38; Get-Content docs/architecture.md | Select-Object -Last 25`
> - Ran ``@'
> # Phase 6: outage recovery and capacity
>
> 24 November 2026. Design specification building on Phases 1–5 and the original brief. No executable application, load test, or real Salesforce capability verification is introduced. All upstream interfaces below are required assumptions, not verified Salesforce features.
>
> ## 1. Decision and feasibility envelope
>
> Use durable replay, a bounded recovery barrier, and the existing PostgreSQL outbox. Keep every original edit even when its pending field application becomes obsolete. Modified-record snapshots cannot recover overwritten values or actors and cannot satisfy the history requirement.
>
> The provisional 120-minute estimate is replaced. Budget each mapped update as one revision read, one complete record-history page through that revision, and one conditional patch: **three requests per job**, before exceptional overhead. Cached coverage, creates requiring only one call, already-agreed values, and obsolete fields may improve results; the estimate takes no credit for those savings. More history pages, repeated races, or dependency chains increase the cost.
>
> Planning inputs:
>
> | Input | Assumption |
> |---|---|
> | Original edits | Peak 20/s per direction; quiet 1/s per direction; Phase 1 burst/hot-record bounds retained |
> | Outage | 21,600 seconds; PostgreSQL remains durable and portal saves continue |
> | Outage history | 432,000 Salesforce edits; no sustained integration echoes while our API calls cannot succeed |
> | Boundary uncertainty | At most 1,000 additional replay entries from checkpoint lag, overlap, or pre-outage in-flight applications; measure and replace this input if exceeded |
> | Replay page | Up to 500 entries; arithmetic assumes full pages except the last, stable bounded manifest and completion proof |
> | Local ingestion | At least 200 delivery/manifest applications/s including duplicates, echoes and database work; reserve 100/s for live deliveries and 100/s for replay |
> | Record-history amplification | At most one page per job at the selected revision; revision-associated history available without scanning an unbounded record history |
> | Outbound service | Configured pacing 90/s; assume at least 89 useful request starts/s sustained, allowing pacing/transaction overhead; absolute cap remains 100/s |
> | Extra requests | At most 10/s including repairs, retries, dependencies, stream pages and control calls; explicit allocation below |
> | Replay retention | At least 72 h; mutation/external-ID resolution remains valid for the recovery/retry window |
>
> These throughput floors, page sizes, bounded exceptions and latency limits require adapter confirmation and production measurement. They are conditions for the estimate, not performance guarantees. Account quotas must permit about 7.78 million starts/day at 90/s; a lower daily quota invalidates this model even if the per-second cap permits it.
>
> ## 2. What remains on disk during the outage
>
> Portal saves commit authenticated actor/server time, original facts/fields, both-source candidates, decisions, current projections, request-key responses and individual outbound jobs in one PostgreSQL transaction. Existing mappings and unresolved parent/owner waiters remain durable. A portal response means accepted locally, with pending/blocked remote status.
>
> Known no-effect failures retain RETRY and persisted jitter deadlines. Possibly submitted mutations retain immutable request identity/body/precondition, attempt evidence, UNCERTAIN status and their entity dispatch slot. Never discard them because a timeout, restart, or retry count was reached. Cooldown/effective rate, replay cursor/epoch/target, page manifests and per-entry completion remain durable. Uncommitted transactions have no effect.
>
> Use account-wide outage backoff: after repeated connection failures, pause ordinary mutation dispatch and use a single paced health/recovery probe at no more than one per 60 seconds, respecting longer Retry-After. This refines Phase 5's per-job retries to avoid every pending job probing a dead account. Maximum six-hour probe count is approximately 360 plus detection attempts, each budgeted and recorded; bound initial detection attempts by in-flight capacity. A possibly accepted probe mutation still needs ordinary ambiguity handling; prefer a read-only probe. Successful connectivity starts recovery, not blind backlog dispatch.
>
> Salesforce may continue accepting human edits while its API is unreachable to us. Webhooks that arrive still use apply-before-acknowledge; missing ones remain unknown until replay. Never advance checkpoints on elapsed time or apparent webhook quiet. PostgreSQL storage/WAL survival is assumed; database storage loss requires backup recovery outside this process-restart design.
>
> ## 3. Recovery epoch, replay completion and safe writes
>
> 1. Fence/stop the former egress sender before takeover, wait the Phase 5 full second, restore persisted cooldown and start with no burst credit. Freeze new remote mutations account-wide. Continue portal saves and inbound application. Retain uncertain mutations for reconciliation; read-only discovery may proceed through the shared cap.
> 2. Persist a recovery epoch and a stable target high-water vector H, one boundary per source partition, from the replay adapter. H must include all committed source events before its capture. Keep the earlier committed cursor C for each partition and an available predecessor boundary for overlap. Never subtract from or numerically compare opaque cursors. If overlap is unavailable, resume exactly from C under the adapter contract; if C has expired, stop with a history gap.
> 3. Enumerate stable pages from the overlap boundary through H. Persist each page's identity, predecessor/successor, manifest hash and entry identities before application. Require at most two buffered pages per active partition and backpressure fetching; the capacity example uses one partition. Requests for empty/final pages and high-water discovery count too. Repeat a lost fetch from the saved boundary; never infer completion from an empty transient response.
> 4. Feed every entry to Phase 4's shared transaction. Original facts/actors/times survive, echoes add application evidence, and already-received webhooks complete their manifest entries without another edit. Invalid events or identity/hash mismatches block progress visibly. Independent valid events may proceed, but cannot bridge the checkpoint gap.
> 5. Under the checkpoint lock, advance C only through linked, fully committed pages, atomically with the final entry/completion evidence. Later pages cannot jump an incomplete predecessor. Mark the epoch's inbound history COMPLETE only after every partition has an authoritative terminal proof through H, all manifest entries through H are committed, and contiguous checkpoints reach H. Persist the proof and completion state transactionally. This means history through H is known, not that both replicas already agree or that no post-H edit exists.
> 6. Reconcile outstanding uncertain mutations using stable identity, authoritative application/history evidence and the Phase 5 upstream-resolution contract. A request accepted before outage may appear in replay; resolve it without a new human fact. A request still capable of taking effect blocks that entity until resolved/fenced. Do not let a new mutation bypass it. The numerical example assumes these operations fit the bounded overhead and 50-second non-ingestion allowance; long ambiguity makes recovery longer.
> 7. Release the account mutation barrier after the committed H proof. For every old or fresh job, obtain an observation R and complete history through R, apply that history, then re-evaluate original winner identities under the entity lock. Suppress obsolete unsent fields with explicit outcomes; retain their facts/jobs. Prepare a conditional patch against R only for remaining differences. A source edit after H or after preparation causes history refresh/precondition failure rather than a blind overwrite. A contact still waits for confirmed parent and owner mappings.
> 8. Continue incremental stream replay after H, capturing successive finite high-water boundaries and advancing contiguous checkpoints. This discovers post-H dropped webhooks, including while backlog drains. Cap stream polling interval at five seconds under healthy service; count empty polls. Global stream coverage does not replace per-record revision coverage unless the adapter supplies a trustworthy mapping proof.
>
> For example, portal P changes phone during outage and unseen remote S changes phone and city. Replay audits S and resolves phone under Phase 2; city survives independently. If S wins phone, P's unsent phone field is superseded. If P wins, its conditional repair follows history coverage. If another remote edit arrives after R, the precondition fails and its history is merged before rebuilding. No recovery-time timestamp is assigned to an old human edit.
>
> A global barrier favors auditable safety and simple coordination; it delays fresh remote writes. Per-partition/record early release is a possible later optimization only with equivalent complete coverage proofs. It is not assumed in these calculations.
>
> ## 4. Crash boundaries
>
> | Restart boundary | Durable result and next action |
> |---|---|
> | Before/after portal commit or webhook commit | Rollback partial work, or recover complete fact/projection/job/receipt; retry by original identity |
> | Epoch allocated, H discovery unfinished | Mutation barrier stays closed; finish discovery; do not claim a completion boundary |
> | H committed, page fetch lost | Re-fetch from committed cursor/page links to the same H |
> | Manifest persisted, some entries committed | Reapply incomplete entries; committed receipts deduplicate overlap; no partial-page checkpoint |
> | Later page complete, earlier page incomplete/invalid | Checkpoint remains before gap, even across restart |
> | Last entry commits but completion response lost | Read committed markers/proof; repeat safely; never create another fact |
> | H proof committed, barrier release not observed | Derive release eligibility from committed epoch/proof; conservatively reconcile before mutation |
> | Read/history finishes, mutation preparation not committed | Re-evaluate candidates and observation; no request assumed prepared |
> | Mutation prepared, sent, or remotely accepted without local success commit | UNCERTAIN; keep entity slot, resolve exact identity; never blindly reconstruct a different mutation |
> | Local success committed, echo arrives later | Verify application evidence; no duplicate original edit or return loop |
> | Dispatcher dies or lease expires | Confirm termination/fencing before replacement, wait one second, honor cooldown; lease expiry alone cannot authorize egress |
> | Database commit unavailable | Stop acknowledgment/dispatch; no volatile queue substitutes for committed history |
>
> If the process was down as well, extend the replay target/volume to the actual coverage gap. A six-hour API outage plus additional downtime is not a six-hour recovery input.
>
> ## 5. Full peak-load arithmetic
>
> ### Stage A: recover the missing inbound history
>
> Original outbound jobs: `20 × 21,600 = 432,000`.
>
> Missing inbound entries including bounded overlap: `432,000 + 1,000 = 433,000`. Replay pages: `ceil(433,000 / 500) = 866`. Allow four more discovery/terminal/control calls: **870 calls**. If partitions or API semantics require more, add them explicitly.
>
> Replay can use up to 5 starts/s while ordinary writes are paused: fetch lower bound `870 / 5 / 60 = 2.90 minutes`. Local replay processing is the bottleneck: `433,000 / 100 / 60 = 72.17 minutes`. Fetch and ingest overlap, with buffer backpressure. Add 50 seconds for bounded discovery, first/last-page latency, terminal proof, egress restart pause and initial ambiguity resolution: `(4,330 + 50) / 60 = 73 minutes`. One serial page fetch every five seconds still supplies 100 entries/s with full pages; extra latency/short pages reduce throughput. The 50 seconds is an explicit aggregate assumption, not a consequence of the request cap.
>
> Fresh portal edits accumulated during the barrier: `20 × 4,380 = 87,600`. Jobs when barrier releases: `432,000 + 87,600 = 519,600`. Salesforce also generates 87,600 new original edits during this stage; they are post-H live/incremental-replay traffic, not silently included in the fixed 433,000 entries. With writes paused, live human deliveries plus one duplicate each require at most 40 applications/s. Reserved live capacity of 100/s covers this and bounded outstanding echoes; incremental stream recovery must also be scheduled if webhooks remain missing.
>
> ### Stage B: drain outbound jobs while new edits continue
>
> All classes share the sole 90/s paced dispatcher; use a conservative **89/s sustained service floor** in the arithmetic:
>
> | Request class | Starts/s budget | Meaning |
> |---|---:|---|
> | Ongoing portal jobs | 60 | 20 jobs/s × (read + history page + patch) |
> | Conflict/side-effect repairs | 2 | Up to 2/3 complete three-call repair workflows/s |
> | Retries/ambiguous resolution | 2 | Additional attempts, status/read/history calls and failed preconditions |
> | Mapping/dependency extras | 1 | Extra lookups/parent resolution beyond each edit's baseline |
> | Incremental stream history | 1 | Up to 500 entries/s at full pages; includes overlapping echo history |
> | Discovery/control | 1 | Boundary/empty polling and operational account reads |
> | Exceptional overhead reserve | 3 | Additional pages or transient burst work; do not count it twice |
> | Old-job service | 19 | Remaining requests, approximately 6.333 three-call jobs/s |
> | Capacity margin below configured pacing | 1 | Allows rounding/persistence/scheduling loss; unused capacity is not burst credit |
> | Total | 90 | Every outbound attempt is represented |
>
> Net job drain: `(89 - 60 - 10) / 3 = 19 / 3 = 6.333 jobs/s`.
>
> Drain time: `519,600 × 3 / 19 / 60 = 1,367.37 minutes`.
>
> Total from connectivity restoration: `73 + 1,367.37 = 1,440.37 minutes`, approximately **24 hours**, under these conservative inputs. At an ideal achieved 90/s with no service margin the same model gives `73 + 519,600 × 3 / 20 / 60 = 1,372 minutes` (22 h 52 min). Neither is the earlier 120-minute result.
>
> At the conservative drain rate, baseline mutation attempts are about `20 + 6.333 = 26.333/s`. Add up to 0.667 repair mutations/s: about 27 echoes/s plus 20 human source edits/s = 47 source events/s. With one duplicate per live event, about 94 live applications/s fit the reserved 100/s. Incremental replay at 47 entries/s fits the other 100/s local processing allocation and requires approximately `47 / 500 = 0.094` full page requests/s, plus bounded overlap/empty polls within its 1/s budget. Phase 1's 200/s ingestion assumption therefore remains feasible on paper. More integration side effects, duplicates or short pages require recalculation. Even if post-H webhooks are all dropped, a 100/s replay service exceeds the approximately 47/s source stream, so the post-H backlog can drain under these bounds.
>
> Total Stage B requests at saturation are about `89 × 82,042.11 = 7,301,747`, including about 4.92 million for ongoing portal jobs, 1,558,800 for the fixed old jobs and 820,421 overhead calls. Add Stage A discovery/pages and other actual Stage A calls separately; unused reservations are not fictional requests. In particular Stage A incremental post-H replay, probes, ambiguity resolution and controls are additional actual calls within the shared budget, not part of the 870 fixed-boundary calls.
>
> The dominant bottleneck is outbound request capacity, especially two coverage calls per mutation. Fetching 866 pages is not the dominant cost. Storage and local throughput may become bottlenecks if their assumptions fail.
>
> ## 6. Scheduling, latency and sensitivity
>
> Implement weighted fair scheduling of request descriptors, not independent rate buckets. Over recurring service rounds reserve the table's shares, carry incomplete workflow progress durably, select oldest eligible entity within each class, and lend idle shares to other ready classes. Enforce per-record dispatch ordering and parent dependencies. Give live inbound work 100/s local processing and replay 100/s; independent entities progress despite a poisoned replay entry, but completion cannot bypass that entry.
>
> During Stage A, mutation traffic is gated, so only eligible reads/replay/control consume requests. During Stage B, fresh portal workflows receive 60/s and old jobs at least 19/s under the overhead bounds. Repair/ambiguity and dependency classes receive positive service. Bursts use slack or briefly queue; rotate fairly among tenants/entities so a hot record cannot monopolize a class. The table is a sustained-load service model; an implementation must bound short-round scheduling delay and connection usage before claiming a worst-case latency. Any overhead above its allocation reduces old-job service and triggers an updated ETA rather than being discarded.
>
> Catch-up has a separate target. Do not promise fresh traffic within 30 seconds while the global barrier is closed, during backlog drain, or while history polling is recovering gaps. Declare recovery complete only after H coverage, no unresolved pre-outage uncertain mutations or unresolved required historical gaps, old jobs have recorded outcomes, incremental replay is current through a recently captured boundary, dependencies/repairs are resolved, and queues fit healthy latency bounds. Blocked/permanent failures remain visible and prevent a whole-account synchronized claim; an unaffected subset may report its own status explicitly.
>
> Healthy inbound allocation remains `5 delivery + 10 queue/lock + 2 apply/commit + 3 visibility = 20 seconds`. For the explicit three-call outbound workflow, revise Phase 5's two-call example to `2 portal commit + 5 total queue/pacing/wait + 15 sequential API completion + 3 visibility = 25 seconds`, leaving 5 seconds margin. The five-second queue allocation includes waits between all three calls and bounded dependencies. With 20/s healthy fresh work using 60 requests/s and overhead <=10/s, 89/s service leaves 19/s margin. An extra 100 portal edits in a burst need up to 300 requests; the baseline margin alone would take `300 / 19 = 15.79 seconds` to clear them, so these averages **do not prove** the five-second queue bound.
>
> To retain Phase 1's burst envelope and worst-case target, assume healthy per-call completion <=2 seconds (stricter than Phase 5's five-second timeout) and allow aggregate queue/wait <=20 seconds: `2 + 20 + 6 + 2 = 30 seconds`, with destination visibility tightened to <=2 seconds. This is an alternative conditional operating envelope, not an established guarantee; it also requires scheduling/connection tests proving <=20 seconds total wait for every burst workflow, no extra pages/races and resolved dependencies. Otherwise require a smaller admitted burst/stronger combined history interface or disclose that worst-case 30 seconds is unmet. Keep the timeout at five seconds for failure detection, but a call exceeding the healthy bound already invalidates that latency claim. Do not call the sustained throughput calculation a latency proof.
>
> At 89 starts/s and five-second in-flight calls, provision at least `ceil(89 × 5) = 445` concurrent connection/request slots plus operational headroom (e.g. 450); no entity has concurrent unresolved mutations. Database and dispatcher must sustain the corresponding durable accounting. With fewer connections the achieved rate falls; measure it before using this ETA.
>
> For changed inputs, use `T_drain_minutes = a × B / (R - a × p - E) / 60`, only when denominator is positive. Here a=3 calls/job, B=519,600, p=20 new jobs/s, E=10 extra calls/s, R=89. Add barrier time; also recompute B from barrier duration and p.
>
> | Changed sustained condition after the same barrier | Consequence |
> |---|---|
> | Throttle to R=70 | `70 - 60 - 10 = 0`: no old backlog drain; preserve work, honor cooldown, escalate |
> | R=80 | `519,600 × 3 / 10 / 60 = 2,598 min` drain, plus 73 min barrier |
> | Portal p=25, fixed initial B for sensitivity only | `89 - 75 - 10 = 4`; drain `519,600 × 3 / 4 / 60 = 6,495 min`; a true six-hour 25/s outage also increases B |
> | Two history pages per job, a=4 | `89 - 80 - 10 = -1`: unstable even before draining old work |
> | Quiet workload, 1/s each direction | Outage jobs 21,600; history 22,600 including overlap; 46 fixed pages; local replay 226 s plus 50 s = 4.6 min; B=21,876; drain `21,876 × 3 / (89 - 3 - 10) / 60 = 14.39 min`; total about 19 min |
>
> At p=20 and E=10, stability requires `a < (89 - 10) / 20 = 3.95`; at a=3 it requires `p < 26.33 edits/s`. Higher rates, repeated contention, 429 cooldowns, unrecoverable mappings or poison history invalidate the ETA and may prevent convergence. Recompute remaining duration from observed request/job rates; never exceed the cap to meet an old estimate.
>
> ## 7. Storage and retention sizing
>
> Illustrative binary units: KiB=1,024 bytes, GiB=1,024^3. Row allowances include associated fields/indexes as stated; they are sizing assumptions to validate with actual schema and distributions, not measured PostgreSQL sizes. Mean two changed fields/edit, <=64 KiB event bound retained; maxima and long values can be much larger than averages.
>
> | Six-hour durable content | Count and assumed allowance | Approximate GiB |
> |---|---|---:|
> | Both-direction original facts/fields | 864,000 × 1 KiB | 0.824 |
> | Decisions and audit lookup indexes | 864,000 × 1 KiB | 0.824 |
> | Portal request/outbox/field outcomes | 432,000 × 1.5 KiB | 0.618 |
> | Event receipts/application evidence | 864,000 × 0.5 KiB, allowing 432,000 human events plus 432,000 eventual propagation events | 0.412 |
> | Mutation bodies and baseline attempt/accounting evidence | 432,000 × 3 KiB, including three request records/job | 1.236 |
> | Dependency/replay/control allowance | Fixed provisional reserve, including bounded page buffers/manifests | 0.250 |
> | Total incremental live-data envelope | Excludes WAL/replicas/backups and unrelated existing data | **4.164** |
>
> During a complete webhook failure remote facts/receipts are not yet local; that part of the envelope is filled during recovery. Duplicate delivery does not create another original fact or receipt for the same event, but consumes processing and may add bounded diagnostics. Ambiguous attempts and repairs add storage beyond the baseline; budget from actual counts. Up to 360 outage probe records at 1 KiB add less than 0.001 GiB; initial detection calls remain separately counted.
>
> For five million existing entities assume 6 KiB/entity for projections, IDs, candidates/resolutions, dispatch/evidence and indexes: `5,000,000 × 6 / 1,048,576 = 28.61 GiB`. This explicitly includes up to nine business fields per record with two-source metadata; inspect actual row sizes rather than equating record count to history volume.
>
> Peak daily growth is 3,456,000 original edits (40/s), 1,728,000 portal jobs and, during catch-up, up to about 4,060,800 source events/day (47/s), including propagation. Use 2 KiB/original for fact+decisions, 4.5 KiB/portal job for outbox/request+mutation/three attempts, and 0.5 KiB/source event for receipt/evidence. This gives `6.592 + 7.416 + 1.936 = 15.944 GiB/day`, before extra requests. The full 10/s overhead at 1 KiB additional attempt/evidence per request adds `864,000 / 1,048,576 = 0.824 GiB/day`: approximately **16.77 GiB/day**. Event buffers/control reserve and long fields need separate headroom. This is conservative for steady state but may underestimate unusually large payloads.
>
> For six hours plus approximately 24-hour recovery (30 hours), incremental live data is about `16.77 × 30 / 24 = 20.96 GiB`. Example initial storage envelope: `(28.61 + 20.96 + 0.25) × 2 = 99.64 GiB`, allowing a factor of two for bloat/free working space, **plus** separately provisioned WAL/temporary storage (illustratively another 50 GiB, to measure), plus historical data already retained, backups and replicas. This is a roughly 150-GiB initial primary-disk planning envelope, not a fixed production size or a storage guarantee. Every additional retained peak day adds approximately 16.77 GiB logical data before headroom. Never prune required history merely to fit the example disk.
>
> 72-hour upstream replay retention covers a six-hour outage plus about 24-hour recovery, leaving about 42 hours margin. Retention must apply to the oldest uncommitted cursor and every required partition, including records blocked by poison entries. Trigger intervention when `remaining upstream retention < estimated remaining recovery + 6 hours safety`, or immediately on a detected gap/no positive drain. This recovery-specific guard does not select Phase 7's exactly three primary operational metrics. Persist dedup/mutation identities for their full retry/replay horizon; local audit retention is a separate client policy and is not 72 hours. If upstream history expires, snapshot repair can restore limited current state, but mark missing historical edits/actors unrecovered and require archive/integration access or an explicit requirement change.
>
> ## 8. Acceptance review and open decisions
>
> Manual review traces the restart table, concurrent remote edit example, contiguous replay proof, parent/uncertain-write gates, rate allocation, echo amplification and storage growth against Phases 2–5. Arithmetic is checked separately; no production throughput or executable recovery tests are claimed. The full request model admits eventual drain under the stated bounded workload and upstream contracts. Snapshot-only history recovery is explicitly rejected. The 120-minute estimate is superseded by about 24 hours at the conservative service floor.
>
> Remaining client/production gates: Q03/Q04 stable complete replay, page sizes/partition completion and retention; Q08/Q09 safe create, bounded revision-associated history and outstanding mutation resolution; Q10 attributable echoes/side effects; Q11/Q12 dependency resolution; Q14 achieved throughput, quotas, network cap semantics and amplification; Q15 catch-up latency policy; Q17 measured storage/durability and audit retention. The original 30-second worst-case target under the allowed burst remains a feasibility gate requiring the stricter healthy envelope and measured scheduling bounds above, or an agreed contract/workload change. This limitation is recorded rather than presented as verified compliance. Phase 7 must carry it into the consolidated design/open questions. Authentic full chat export remains pending; existing chat files and git commits are untouched. Phases 7–8 are not implemented here.
> '@ | Set-Content -Encoding utf8 docs/recovery-capacity.md``
>
> </details>
>
> The draft now estimates about 24 hours of recovery under the stated load: roughly 73 minutes to replay missing history, followed by outbound draining while new edits continue. Reads and history requests account for most of the increase from 120 minutes.
>
> The capacity review also exposed an unresolved latency issue: average throughput alone does not prove the 30-second worst case during the allowed burst. I’ve recorded the stronger timing assumptions and validation needed before making that claim. I’m checking the arithmetic and cross-document consistency now.
>
> <details><summary>Ran 3 commands</summary>
>
> - Ran `@'
> import math
> b=432000+20*4380
> drain=b*3/19/60
> print({'backlog':b,'replay_pages':math.ceil(433000/500),'replay_minutes':433000/100/60,'drain_minutes':drain,'total_minutes':73+drain,'stage_b_calls':89*drain*60,'quiet_total_minutes':4.6+21876*3/76/60})
> parts=[864000/1048576,864000/1048576,432000*1.5/1048576,864000*.5/1048576,432000*3/1048576,.25]
> daily=(3456000*2+1728000*4.5+4060800*.5+864000)/1048576
> print({'six_hour_gib':sum(parts),'daily_gib':daily,'base_gib':5000000*6/1048576,'30h_gib':daily*30/24,'primary_plus_wal_gib':(5000000*6/1048576+daily*30/24+.25)*2+50})
> '@ | python -`
> - Ran `git diff --check; rg -n '120|Phase 6|30.second|10 requests' docs/assumptions.md IMPLEMENTATION_PLAN.md; git status --short`
> - Ran ``@'
> from pathlib import Path
> p=Path('docs/assumptions.md')
> s=p.read_text(encoding='utf-8')
> s+='''
>
> ## Phase 6 refinements (supersede preliminary capacity/timing estimates)
>
> `recovery-capacity.md` now governs recovery accounting. A conservative mapped-update workflow costs three requests (revision read, complete history page, conditional patch), not one. At 20 portal edits/s ongoing baseline is 60 requests/s, plus at most 10/s exceptional overhead. Operate at 90/s but assume 89/s sustained achieved service for planning. A 73-minute global replay barrier raises the 432,000-job outage backlog to 519,600; draining at 19/3 jobs/s takes 1,367.37 minutes. Total estimated recovery is 1,440.37 minutes (about 24 hours), superseding the provisional 120 minutes. The 72-hour replay assumption now provides about 42 hours margin after six hours outage plus recovery, rather than the original provisional 64 hours.
>
> Q03/Q04 require the explicit partition high-water/completion proofs, bounded 500-entry paging and <=1,000 overlap/in-flight entries described there. Q14 additionally requires the three-call workload, <=10/s extra requests, >=89/s actual dispatch throughput, sufficient concurrent requests and account daily quota. Q15 is resolved provisionally as no fresh-traffic 30-second guarantee during replay or backlog catch-up; resume only after committed recovery/queue checks. Q17 gains explicit storage allowances and a retention-risk intervention rule. All remain assumptions pending confirmation/measurement.
>
> The earlier one-/two-call latency allocation cannot prove the worst-case target for three sequential calls plus Phase 1's burst. The Phase 6 review explicitly leaves that feasibility gate open: either validate its stricter <=2-second healthy call/visibility bounds and <=20-second aggregate workflow queue bound, or agree a smaller burst/stronger combined interface. The five-second attempt timeout remains failure detection, not a healthy latency promise. Do not cite earlier allocations as unconditional compliance. Account-wide outage probing/backoff refines per-job retries; all attempts still share the Phase 5 cap. The existing source conflict/history rules are unchanged.
> '''
> p.write_text(s,encoding='utf-8')
> p=Path('IMPLEMENTATION_PLAN.md');s=p.read_text(encoding='utf-8')
> s=s.replace('| 6 | Outage recovery and capacity | 1–5 | Not started |','| 6 | Outage recovery and capacity | 1–5 | Complete |')
> needle='6. Schedule fresh traffic and backlog'
> pos=s.index(needle)
> s=s[:pos]+'''   Phase 6 result: the one-call example above is superseded by `docs/recovery-capacity.md`. Three calls per mapped update, 89/s achieved service within 90/s pacing, 10/s exceptional overhead and a 73-minute replay barrier yield approximately 1,440.37 minutes (24 hours) recovery. Worst-case healthy burst latency remains an explicit feasibility gate; see the Phase 6 timing assumptions.
> '''+s[pos:]
> needle='| 24 November 2026 — Phase 5 |'
> pos=s.index(needle)
> s=s[:pos]+'''| 24 November 2026 — Phase 6 | Created `docs/recovery-capacity.md`: six-hour durable state, account outage probing, recovery epochs/partition high-water proofs, bounded overlap/manifests/contiguous checkpoints, global replay barrier and per-record history-covered conditional release, restart table, full peak/quiet request and echo budgets, fair scheduling, latency limitations, sensitivity, connection sizing, storage and retention. Updated `docs/assumptions.md` to supersede preliminary estimates. | Checked arithmetic using Python, manually traced crash/overlap/conflict/ambiguity/dependency cases against Phases 2–5 and the brief, and ran diff whitespace checks. Design acceptance met under explicit upstream/load/service assumptions: replay does not rely on snapshots, calls share the cap, estimates are conditional, and 19/3 old jobs/s net service is positive. Recovery estimate is 1,440.37 minutes, not 120; peak storage allowance is about 16.77 GiB/day logical growth. Healthy 30-second worst-case burst scheduling remains unproven and explicitly gated on stronger timing/measurement or contract changes. Q03/Q04/Q08–Q15/Q17 remain provisional. No executable tests, production benchmark, real Salesforce verification, code, chat-history edits or git commits; Phases 7–8 remain unstarted. |
> '''+s[pos:]
> p.write_text(s,encoding='utf-8')
> '@ | python -
> git diff --check
> git diff --stat
> git status --short``
>
> </details>
</details>

Phase 6 is complete as design work. The main artifact is [docs/recovery-capacity.md](./docs/recovery-capacity.md).

It covers:

- **Durability during the outage:** which edits, audit facts, pending jobs, uncertain requests, retry deadlines and checkpoints remain on disk.
- **Missing-history recovery:** replay from committed checkpoints, deduplicate overlapping deliveries, and require explicit completion proofs for every partition. Snapshots cannot recover overwritten edits or their actors.
- **Safe backlog processing:** pause mutations while recovering missing history, then refresh each record’s history and revision before conditional writes. This prevents stale queued changes from blindly overwriting unseen remote edits.
- **Restart handling:** recovery actions for crashes during page processing, checkpoint advancement, request preparation, remote acceptance and dispatcher failover.
- **Capacity and scheduling:** shared request allocations for fresh work, backlog, reads, history pages, retries, repairs and dependencies, plus throttling and increased-load scenarios.
- **Storage and retention:** estimates for current records, history, jobs, request evidence, disk headroom and replay-retention risk.

The full accounting replaces the preliminary **120-minute estimate with approximately 24 hours**:

| Step | Estimate |
|---|---:|
| Replay missing inbound history | 73 minutes |
| Outbound jobs after replay | 519,600 |
| Drain while new edits continue | 1,367.37 minutes |
| Total recovery | **1,440.37 minutes** |

The increase comes mainly from budgeting **three calls per mapped update**—revision read, history page and conditional patch—rather than one write. These are conditional planning estimates, not measured Salesforce performance.

The review also identified an unresolved requirement: average capacity does **not** prove the 30-second worst case during the assumed burst. The document records the stronger timing assumptions and scheduling validation needed before claiming compliance.

I updated [docs/assumptions.md](./docs/assumptions.md) and the phase status/execution log in [IMPLEMENTATION_PLAN.md](./IMPLEMENTATION_PLAN.md). Arithmetic was checked with Python, recovery cases were manually traced against Phases 2–5, and whitespace checks passed. No executable tests or real Salesforce verification were performed. Chat history and commits were preserved; Phases 7–8 remain unstarted.

# Execute phase 7

> proceed with phase 7

<details><summary>6 previous messages</summary>

> I’ll read Phase 7 and check its dependencies and existing artifacts, then carry out the authorized work and record the validation results in the plan.
>
> <details><summary>Explored 2 files, ran 9 commands</summary>
>
> <details><summary>Explored a file</summary>
>
> - Read `./IMPLEMENTATION_PLAN.md`
>
> </details>
>
> - Ran `git status --short; rg --files -g AGENTS.md -g package.json -g '*phase*' -g '*Phase*'`
>
> - Ran `Get-Content Round1-Tequity.html; Get-Content docs/requirements-checklist.md; Get-Content docs/architecture.md`
>
> - Ran `Get-Content docs/assumptions.md; Get-Content docs/recovery-capacity.md`
>
> - Ran `Get-Content docs/conflict-rules.md; Get-Content docs/inbound-flow.md; Get-Content docs/outbound-flow.md; Get-Content docs/data-model.md`
>
> - Ran `git log --oneline -12; rg --files -g '!Round1-Tequity.html' -g '!*.pdf'; Get-Content C:/Users/yashp/.codex/skills/.system/openai-docs/SKILL.md`
>
> - Ran `Get-Content Round1-Tequity.html | Select-Object -Skip 700 -First 350`
>
> <details><summary>Explored a file</summary>
>
> - Read `./docs/recovery-capacity.md`
>
> </details>
>
> - Ran `Get-Content docs/conflict-rules.md | Select-Object -First 105; Get-Content Chat_History/chat_history.md -TotalCount 28; Get-Content docs/requirements-checklist.md`
>
> - Ran `$brief = Get-Content Round1-Tequity.html -Raw; $body = [regex]::Match($brief, '(?s)<main.*?</main>').Value; [System.Net.WebUtility]::HtmlDecode([regex]::Replace($body, '<[^>]+>', ' '))`
>
> - Ran `Get-Content docs/recovery-capacity.md | Select-Object -Skip 155; Get-Content docs/assumptions.md | Select-Object -First 100; Get-Content docs/conflict-rules.md | Select-Object -First 42; Get-Item Chat_History/chat_history.md | Select-Object Length; rg '^#' Chat_History/chat_history.md`
>
> </details>
>
> Phases 1–6 have the required design artifacts. I’ll carry forward the revised recovery estimate of about 24 hours and the unresolved 30-second latency feasibility gate. The existing chat-history file does not establish complete coverage of every task chat, so that submission item will remain pending.
>
> I’ll create the consolidated design, 10-minute walkthrough, and submission checklist. I’m also using the openai-docs skill to check export guidance; existing chat files and commits will stay untouched.
>
> <details><summary>Created a file, ran 3 commands, searched the web</summary>
>
> Searched the web for `site.help.openai.com export ChatGPT history data export`
>
> - Ran `Get-Content docs/recovery-capacity.md | Select-Object -Skip 121 -First 35; Get-Content docs/architecture.md -TotalCount 95; Get-Content docs/inbound-flow.md | Select-Object -First 55; git remote -v; git ls-files`
>
> Searched the web for `https://help.openai.com/en/articles/7260999-exporting-your-chatgpt-history-and-data`
>
> - Ran `Get-Content docs/recovery-capacity.md | Select-Object -Skip 112 -First 12; Get-FileHash Chat_History/chat_history.md -Algorithm SHA256; git rev-parse HEAD; git rev-list --count HEAD`
>
> - Wrote `./docs/design.md` (+160 -0)
>
> - Ran `rg -n 'spacing|millisecond|ceil|90|1/C|1000' docs/outbound-flow.md`
>
> </details>
>
> The draft now covers all 12 sections from the brief and selects exactly three operational metrics. The consistency review preserves the key distinction: verified echoes stop without creating new edits, while genuine remote changes can still trigger repairs. Recovery uses the same rules after replay completes.
>
> I’ll finish the walkthrough and checklist, then check links, arithmetic, repository state, and the existing chat file’s hash.
>
> <details><summary>Created 2 files, edited 3 files, ran 2 commands</summary>
>
> - Wrote `./docs/design.md` (+1 -1)
> - Wrote `./docs/submission-checklist.md` (+51 -0)
> - Wrote `./docs/walkthrough.md` (+19 -0)
> - Wrote `./IMPLEMENTATION_PLAN.md` (+2 -1)
> - Wrote `./docs/requirements-checklist.md` (+18 -0)
> - Ran `$files = @('docs/design.md','docs/walkthrough.md','docs/submission-checklist.md','docs/requirements-checklist.md'); foreach ($file in $files) { $content = Get-Content -LiteralPath $file -Raw; foreach ($match in [regex]::Matches($content, '\]\(([^)]+)\)')) { $target = $match.Groups[1].Value; if ($target -notmatch '^https?://') { $resolved = Join-Path (Split-Path $file) $target; if (-not (Test-Path -LiteralPath $resolved)) { throw "Broken link in ${file}: $target" } } } }; $design = Get-Content docs/design.md -Raw; 'Design numbered sections: ' + [regex]::Matches($design, '(?m)^## \d+\.').Count; 'Metric rows: ' + [regex]::Matches($design, '(?m)^\| [123]\. ').Count; 'Walkthrough seconds: ' + (90+50+50+90+60+50+120+50+40); 'Peak total minutes: ' + (73+519600*3/19/60); 'Quiet total minutes: ' + (4.6+21876*3/76/60); 'All local links resolve'; Get-FileHash Chat_History/chat_history.md -Algorithm SHA256; git diff --check; git status --short; git log --oneline -3`
> - Ran `$design = Get-Content docs/design.md -Raw; $metrics = [regex]::Match($design, '(?s)## 11\..*?(?=## 12\.)').Value; $count = [regex]::Matches($metrics, '(?m)^\| [123]\. ').Count; if ($count -ne 3) { throw "Expected three primary metrics; found $count" }; 'Primary metric rows in section 11: ' + $count; foreach ($file in @('docs/design.md','docs/walkthrough.md','docs/submission-checklist.md')) { if (Select-String -LiteralPath $file -Pattern '[ \t]+$') { throw "Trailing whitespace: $file" } }; 'New artifact whitespace checks passed'; git diff --stat; git rev-parse HEAD`
>
> </details>
</details>

Phase 7 artifacts are ready:

- [Design](./docs/design.md): all 12 sections and exactly three operational metrics.
- [10-minute walkthrough](./docs/walkthrough.md).
- [Submission checklist](./docs/submission-checklist.md), including [official export guidance](https://help.openai.com/en/articles/7260999-exporting-your-chatgpt-history-and-data).

Validated links, arithmetic, section counts and whitespace. Chat history and existing commits remain unchanged.

Phase 7 is recorded as **partial**: complete authentic chats, final repository packaging/access validation and revised delivery timing remain pending. The 30-second latency feasibility gate stays explicit; Phase 8 was not started.

> explain in detail what all was implemented

Phase 7 assembled the **required submission documents** from the design work completed in Phases 1–6. It did **not implement the executable synchronization service**; that remains optional Phase 8.

The work produced three documents, updated two project records, and reviewed the design for consistency.

### 1. Consolidated design document

Created [docs/design.md](./docs/design.md).

This brings the earlier specifications into one document following **all 12 sections required by the original assignment**.

| Section | What it contains |
|---|---|
| **1. Assumptions and questions** | Workload, payload, replay, conflict, mapping, durability and API assumptions; consequences if those assumptions are false. |
| **2. Systems and interfaces** | A diagram showing users, Salesforce, our HTTP role, worker role and PostgreSQL; who initiates each HTTPS call; proposed interface contracts. |
| **3. Subproblems and build order** | Why identity and conflict rules come first, followed by durable transactions, inbound handling, outbound work, dependencies and recovery. |
| **4. Stored data** | Current records, source mappings, field candidates, immutable history, receipts, jobs, mutations, evidence, dependencies and replay checkpoints. |
| **5. Inbound synchronization** | Validation, duplicate detection, history preservation, field resolution and commit-before-acknowledgment. |
| **6. Outbound synchronization** | Atomic portal save/outbox, worker processing, conditional patches, create recovery, retries, throttling and request pacing. |
| **7. Concurrent changes** | How independent fields survive and how same-field conflicts resolve deterministically. |
| **8. Duplicate and older events** | Separate rules for duplicate deliveries, distinct equal-value edits and late events affecting different fields. |
| **9. Loop prevention** | How verified echoes become application evidence while genuine remote edits and necessary repairs still proceed. |
| **10. Six-hour outage** | Durable state, replay barriers, checkpoint safety, request accounting and recovery arithmetic. |
| **11. Operational metrics** | Exactly three primary metrics, escalation thresholds and reasons. |
| **12. Exclusions and submission state** | What remains outside scope, the distinction between demo and production persistence, and outstanding submission items. |

The document links to the detailed earlier specifications so a teammate can move from the consolidated explanation to schema, pseudocode and failure cases.

### 2. Conflict, duplicate and loop rules were made explicit

These rules were **consolidated and reviewed**, rather than newly implemented in code.

**For conflicts**, each field retains the latest original edit from each source, selected by that source’s revision. Those two candidates are then compared by timestamp, with the portal winning exact timestamp ties.

For example:

- A portal city change and a Salesforce phone change both survive.
- Two edits to phone produce one current value, while both original edits remain in history.
- An older event can still contribute a city change even if its phone change is obsolete.

Keeping both source candidates matters when a source clock moves backward. Keeping only the overall current winner would lose information needed for later decisions.

**For duplicates**, the same event identity and canonical content produce no additional business effect. A different source commit that happens to set the same value remains a distinct historical edit.

**For loops**, a verified echo refers back to the original edit and records that it was applied remotely. It does not become another human edit with a newer timestamp. However, a genuine Salesforce change can still require a repair when it differs from the resolved winner.

### 3. Recovery arithmetic was carried forward accurately

The consolidated design preserves Phase 6’s revised estimate of **approximately 24 hours of recovery**, replacing the preliminary 120-minute example.

The calculation assumes:

| Input | Value |
|---|---:|
| Portal-origin edits | 20/s |
| Salesforce-origin edits | 20/s |
| Outage | Six hours |
| Initial portal backlog | 432,000 jobs |
| Replay entries including bounded overlap | 433,000 |
| Replay barrier | 73 minutes |
| Portal jobs when that barrier releases | 519,600 |
| Requests per mapped update | Three |
| Achieved outbound service | 89 requests/s |
| Ongoing portal demand | 60 requests/s |
| Exceptional overhead | 10 requests/s |

That leaves **19 requests/s** for old jobs:

```text
Drain time = 519,600 × 3 ÷ 19 ÷ 60
           = 1,367.37 minutes

Total      = 73 + 1,367.37
           = 1,440.37 minutes
           ≈ 24 hours
```

The document clearly labels this as a **conditional estimate**. More history pages, lower quotas, throttling or repeated conflicts can increase it or prevent the backlog from draining.

It also preserves the unresolved limitation: the assumed workload and sustained throughput **do not prove worst-case healthy synchronization within 30 seconds**, especially during bursts.

### 4. Exactly three operational metrics were selected

The design now specifies these three primary metrics:

| Metric | Escalation |
|---|---|
| **Oldest unconfirmed propagation age** | Page at **30 seconds** in healthy operation; warn at 20 seconds sustained for 60 seconds. |
| **Maximum outbound starts in a rolling second** | Page immediately above **100**; investigate above the configured operating limit of 90. |
| **Recovery retention safety margin** | Page below **six hours**, or on an unsafe recovery estimate, confirmed history gap or expired cursor. |

Each metric includes its purpose, measurement limitations and proposed response. These are proposed operational policies; no monitoring system was built.

### 5. Ten-minute walkthrough was prepared

Created [docs/walkthrough.md](./docs/walkthrough.md).

It allocates **exactly 600 seconds** across:

- Assumptions and open questions.
- Architecture and callers.
- Stored data and build order.
- Conflict handling.
- Duplicate, older-event and loop rules.
- Outbound safety.
- Outage recovery and arithmetic.
- Metrics.
- Exclusions and submission gaps.

It provides talking points and references for follow-up questions. A live presentation or timed rehearsal was **not** performed.

### 6. Submission readiness was documented honestly

Created [docs/submission-checklist.md](./docs/submission-checklist.md).

It records which deliverables exist and which remain pending.

For chat history, it:

- Records the existing file’s size and SHA-256 preservation baseline.
- Explains the available official export process.
- Describes how to verify coverage across all assignment chats.
- Leaves authenticity and completeness **pending**.
- Makes no claim that a summary or reconstructed conversation is a full export.

For git history, it:

- Lists the three existing reachable commits and their messages.
- Records the configured repository origin.
- Notes that current artifacts are not all committed.
- Distinguishes local repository availability from verified recipient access.
- Preserves existing commits without creating, squashing or rewriting any.

It also records that the original submission deadline had already passed and that revised delivery timing remains unresolved.

### 7. Project tracking was updated

Updated:

- [IMPLEMENTATION_PLAN.md](./IMPLEMENTATION_PLAN.md): Phase 7 is **partial**, with an execution-log entry describing work, validation and outstanding gaps.
- [docs/requirements-checklist.md](./docs/requirements-checklist.md): added current evidence against the original requirements while retaining historical phase records.

The pre-existing Phase 6 changes in the assumptions, recovery document and plan were reused.

### 8. Validation performed

The checks confirmed:

- All **12 numbered design sections** are present.
- Section 11 contains **exactly three primary metrics**.
- Walkthrough allocations total **600 seconds**.
- Peak and quiet recovery calculations agree with the earlier capacity model.
- Local document links resolve.
- Whitespace checks pass.
- The existing chat file’s hash is unchanged.
- Git HEAD and the three existing commits remain unchanged.

The duplicate, older-event, echo/repair and outage rules were also manually traced across the specifications.

**No application tests, benchmark, real Salesforce capability verification or executable demo was performed.** Phase 7 remains partial because complete authentic chat coverage and final repository packaging/access and delivery checks are still outstanding.

> what will be done phase 8?

Phase 8 will turn the agreed design into an **optional runnable demo** using **TypeScript on Node.js 20+**, with a fake Salesforce service and durable local storage.

It will demonstrate the synchronization rules rather than connect to real Salesforce.

### What will be built

| Part | Purpose |
|---|---|
| Portal edit handler | Save an edit’s values, actor, time, history and outbound job atomically. |
| Fake Salesforce | Simulate records, revisions, conditional updates, webhooks, replay, outages and ambiguous responses. |
| Inbound handler | Validate events, deduplicate them, preserve history and resolve fields. |
| Outbound worker | Process durable jobs, respect dependencies, retry failures and reconcile uncertain results. |
| Shared request limiter | Enforce the configured cap across writes, reads, retries, repairs and replay calls. |
| Recovery flow | Replay missed changes before safely draining queued writes. |
| Narrated demo command | Run deterministic scenarios and explain their outcomes. |

The demo will require only:

```text
npm install
npm run demo
```

The exact command will be finalized during implementation. It will require no external PostgreSQL, Redis, Docker or Salesforce account.

### Scenarios that will be checked

1. **Duplicate webhook:** repeated delivery produces one logical edit.
2. **Older event after a newer one:** newer fields remain current while every distinct edit stays in history.
3. **Conflicting edits:** different-field changes survive; same-field changes resolve using the documented policy.
4. **Loop prevention and repair:** an echo causes no repeated write, while genuine divergence can still trigger correction.
5. **Parent and child creates:** a contact waits for its customer’s confirmed mapping.
6. **Accepted request with lost response:** recovery avoids duplicate creates and stale overwrites.
7. **Rate enforcement:** all outbound attempts count, including retries; a lower demo cap makes pacing visible.
8. **Outage and restart:** a real process restart preserves pending work while fake Salesforce is unavailable.
9. **Dropped webhook recovery:** replay restores missed edits and their history.

Six-hour outage arithmetic will use **simulated time**, clearly labeled; the demo will not wait six hours.

### Documentation and completion criteria

Phase 8 will also produce a README with setup, execution instructions, limitations and the distinction between **local demo persistence** and **production PostgreSQL**. Meaningful automated checks will verify the behavior, and any justified changes to the design will be recorded.

Completion requires a fresh checkout to run with `npm install` plus one demo command, with the required scenarios passing and restart durability and rate enforcement demonstrated.

**Before starting, Phase 7’s dependency gaps must be inspected.** It is currently partial because authentic chat coverage and final submission packaging remain pending. Those gaps need an explicit direction before proceeding with dependent Phase 8 work.

> what are the phase 7 gaps?

Phase 7 has three outstanding submission gaps:

1. **Complete, authentic AI chat history**
   `Chat_History/chat_history.md` exists and was preserved, but it has not been verified to contain every assignment chat, prompt and reply—including planning and all phases. Complete, unedited source transcripts must be supplied and checked.

2. **Final repository packaging and access**
   Existing commits are preserved, but Phase 6 and Phase 7 artifacts are not all committed. The final submission must include all intended files and full git history, with recipient access verified. No new commits or push were performed.

3. **Revised submission timing and delivery**
   The brief’s deadline was **23 November 2026 at 10:00 AM IST**, which had passed when Phase 7 was performed. Revised timing needs confirmation; nothing has been submitted externally.

The **design, three operational metrics, walkthrough script and submission checklist are ready**.

Separately, real Salesforce capabilities and the worst-case 30-second latency bound remain explicit **design feasibility questions**. They are documented open questions, not missing Phase 7 documents.

Details are in [submission-checklist.md](./docs/submission-checklist.md).

> all three things have been taken care of

<details><summary>2 previous messages</summary>

> I’ll record those three gaps as resolved based on your confirmation, update Phase 7’s status, and keep the production feasibility questions open.
>
> <details><summary>Edited 4 files, ran 2 commands</summary>
>
> - Ran `Get-Content IMPLEMENTATION_PLAN.md | Select-Object -First 65; Get-Content docs/submission-checklist.md; Get-Content docs/requirements-checklist.md | Select-Object -Last 25; Get-Content docs/design.md | Select-Object -Last 6; git status --short`
> - Wrote `./IMPLEMENTATION_PLAN.md` (+2 -1)
> - Wrote `./docs/design.md` (+1 -1)
> - Wrote `./docs/requirements-checklist.md` (+1 -1)
> - Wrote `./docs/submission-checklist.md` (+6 -6)
> - Ran `git diff --check; git diff --stat`
>
> </details>
</details>

Marked **Phase 7 complete** in the plan and submission documents, recording that all three gaps were resolved based on your confirmation.

Phase 8 is ready to begin when you instruct **“proceed with phase 8.”**
