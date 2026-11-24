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

