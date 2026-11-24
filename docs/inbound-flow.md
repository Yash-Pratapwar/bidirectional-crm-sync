# Phase 4: inbound synchronization

24 November 2026. This is an implementation-ready design specification, not executable endpoint code. Dependencies: `assumptions.md`, `conflict-rules.md`, `data-model.md`, and `architecture.md` (Phases 1–3). The original brief supplies HTTPS webhooks and the sample fields; signatures, revisions, replay, provenance, and other additions below are provisional integration contracts, not verified Salesforce features. Outbound dispatch mechanics and recovery scheduling remain Phases 5–6.

## 1. Endpoint and authentication

Salesforce calls `POST /webhooks/salesforce` over HTTPS. The HTTP role authenticates, validates, and applies the event to production PostgreSQL before returning success. It never calls Salesforce inside the request or an open database transaction. A durable raw inbox alone is insufficient for acknowledgment.

Provisional authentication: HMAC-SHA256 over `deliveryTimestamp + "." + exactRawBodyBytes`, using a per-source-account secret selected by a configured key ID. Require a UTC integer delivery timestamp within ±300 seconds of receiver time and a constant-time signature comparison. The key resolves tenant/account authority; compare it with the body rather than trusting body tenant IDs. Accept current and explicitly configured retiring keys during rotation. Bound header sizes; never log secrets or signatures. Reject unsupported content encodings; bound the raw uncompressed JSON body at 64 KiB before parsing. These are contract choices requiring Q13 confirmation, not a claim about native Salesforce signing.

The delivery timestamp is separate from `occurredAt`: a six-hour-old edit can have a fresh authenticated delivery. A sender must re-sign a retried delivery after the timestamp expires. Replays use the authenticated upstream worker adapter and shared semantic validation, not a forged public webhook signature. Timestamp checks limit captured-request reuse; durable event identity prevents repeated effects even inside the signature window. TLS, credential possession, and a fresh signature authenticate the envelope, not the truth of human attribution beyond the trusted source contract.

## 2. Envelope, identity, and validation

Accept one event per request, `Content-Type: application/json`, schema version 1. Parse strictly: reject duplicate JSON keys, invalid UTF-8, unsupported versions, arrays in place of the envelope, unknown envelope/patch keys, and malformed or missing metadata. Parse lossless positive decimal revisions as bigint, never floating-point numbers. Require millisecond UTC `occurredAt` ending in Z. Do not reject an otherwise valid historical edit because its source time is old; visible clock anomalies invalidate timing assumptions but do not justify deleting history.

Required metadata: `tenantId`, `sourceAccount`, `schemaVersion`, `eventId`, `recordId`, `objectType`, `operation`, `sourceIncarnation`, `sourceRevision`, `occurredAt`, stable `actorId`, and nonempty `changedFields`. Retain the sample's `changedBy` as descriptive source attribution; it is not a replacement for stable actor identity. The earlier architecture's optional `actorDisplay` is the normalized internal name for `changedBy`; if both appear, require equality. IDs are opaque bounded strings (1–256 characters); object is exactly Customer or Contact; operation is create or update. Add `editId` for original edits and per-field `provenance` for propagated fields. Require the configured stable source incarnation; revision resets require explicit remapping. Unknown actor-to-portal-principal mappings retain the authenticated stable source actor ID without inventing a human identity.

`eventId` identifies transport delivery content within tenant/account. An original `editId` identifies an immutable logical edit; its namespace includes tenant/account/origin. If absent, derive it deterministically from that namespace and `eventId` only under the contract that replay and webhook preserve the same event identity. The original commit key additionally includes canonical entity, source account/incarnation, origin, and source revision. Different event IDs referring to the same original commit must carry the same logical identity/content; a wrapper change is not another human edit. Same value with a new original commit/identity is a distinct edit.

Compute a versioned canonical content hash after validation: deterministic key order, exact normalized timestamp/revision representation, and explicit null preservation. Include immutable envelope semantics, actor, patch, and provenance; exclude signature, key ID, delivery timestamp, local received time, and replay page/cursor wrappers. A receipt hash verifies event immutability; a separate logical-edit hash excludes transport wrapper identity and includes original edit semantics. Never use a value hash as event identity.

| Object | Allowed patch keys | Provisional validation |
|---|---|---|
| Customer | `name`, `status`, `owner` | Name is nonblank string ≤256 characters. Status is a string from tenant-configured enum; its actual values require Q18 confirmation. Owner is a stable source principal ID. |
| Contact | `fullName`, `phone`, `email`, `city`, `owner`, `customerId` | Full name is nonblank string ≤256 characters; phone ≤64, email ≤320, city ≤256; owner/customer are stable source IDs ≤256. No guessed phone normalization, email matching, or fuzzy identity merge. |

These logical names map explicitly to Phase 3 columns (`fullName` → `full_name`, `customerId` → `customer_entity_id` after resolution). Names and contact customer references cannot be null. Optional phone/email/city and, provisionally, status/owner may be explicitly null; empty strings are rejected in v1 in favor of explicit null clears. A missing key means no edit. Types are string or allowed null only; no coercion, nested objects, or silent truncation. The precise enums/nullability are configuration gates under Q18, not inferred Salesforce rules. A create must include name or fullName/customerId respectively; an update can be a partial patch. Incomplete source creates fail validation rather than invent required values. Unknown updates are handled as placeholders below.

For each propagated field, provenance names `mutationId` and `originalEditId`. Validate against the durable immutable outbound mutation: same tenant/account, entity or safely resolvable external UUID, field/value, original winner identity, and source association. Body-supplied provenance is never trusted merely because it exists. Mixed events require an explicit per-field partition: propagated fields link to existing edits; genuine fields share the original Salesforce commit metadata and are appended as one new logical edit containing only those fields. Side effects require attributable original edit identity/actor/time. Unknown or mismatching claimed provenance is an integrity error; absent provenance denotes a genuine source edit, including edits made by an integration account. If upstream strips required provenance, flag the Q10 capability failure rather than claiming loop safety.

## 3. Atomic application and acknowledgment

All accepted events use the same repository routine, whether delivered by webhook or replay. Under Phase 3's entity/identity locks and unique constraints, perform:

1. Resolve tenant/account and source identity. Acquire identity locks in deterministic order for the event and any referenced identities; allocate canonical identity/mappings only inside this transaction. Lock affected entity rows in sorted UUID order. If new dependencies change the lock set, restart the transaction with the complete set. Never wait on remote I/O while holding locks.
2. Check the receipt key/hash. An identical committed duplicate has no new edit, decision, generation increment, repair, or dependency side effect. For replay, it still marks the matching manifest entry complete in this transaction. A different hash for the same event ID aborts as an integrity conflict.
3. Verify original commit/edit uniqueness and provenance before changes become visible. An alias delivery of an existing logical edit verifies identical content and adds receipt linkage only. Conflicting edit content or two original edit IDs claiming the same commit abort the whole event.
4. Append every new original fact and its explicit fields, including losing, late, and equal-value edits. Propagated fields add application evidence, not new human facts or Salesforce candidates. No partial acceptance of a mixed event.
5. For each newly edited field, replace only the SALESFORCE candidate if its original source revision is greater than that field's stored source revision. Then choose the resolved winner from both source candidates using Phase 2's `(occurredAt, originRank)` rule, with PORTAL winning exact ties. Never discard the whole patch because its record revision is old. Append decision evidence and update typed projections/lifecycle and resolution generation when resolved winner identity/value changes.
6. Persist unresolved reference candidates and indexed dependency waiters. Record source observation/application evidence by remote revision; a late event cannot replace newer observations. A webhook alone cannot establish complete history coverage through a revision: gaps may still exist. `history_covered_through_revision` advances only with the separate complete-history proof.
7. Persist deduplicated reconciliation/repair intent when divergence exists or available evidence cannot safely establish agreement. Key repair intent by entity/resolution generation as in Phase 3. A changed remote observation without a changed local winner can wake/reuse the existing intent; do not multiply equivalent jobs. Previously completed intent must be made due again if new evidence invalidates satisfaction. Use guarded state changes so an active or uncertain mutation is reconciled rather than dispatched concurrently. Before sending anything, the later worker must refresh coverage/revision and recompute the actual patch.
8. Insert accepted receipt with edit/application links and outcome. For replay, atomically complete the manifest entry. Commit all changes, then return 200. Database durability uses Phase 3's synchronous commit assumption. No 202/raw-inbox acknowledgment.

Illustrative pseudocode (repository helpers are specified operations, not implemented functions):

```text
handleWebhook(request):
    context, raw = authenticateAndBoundRawBody(request)
    event = parseValidateNormalize(raw, context)
    try:
        result = applySourceEvent(event, manifestEntry = none)
    catch integrityOrProvenanceConflict:
        recordBoundedIncidentAfterRollbackIfPossible()
        return 409 with stable reason code
    catch temporaryDatabaseOrLockFailure:
        return 503 with Retry-After
    return 200 with committed receipt ID, classification, lifecycle, sync status

applySourceEvent(event, manifestEntry):
    BEGIN
    identity = resolveAndLockIdentityAndDependencies(event)
    receipt = findReceipt(event.scope, event.eventId)
    if receipt exists:
        require receipt.hash == event.hash
        completeManifestEntryIfPresent(manifestEntry, receipt)
        advanceOnlyContiguousCompletedReplayPagesIfEligible()
        COMMIT
        return duplicate(receipt)  // no reapplication of business changes
    parts = verifyCommitIdentityAndClassifyFields(event, identity)
    appendNewOriginalFactsAndFields(parts.original)
    recordVerifiedApplications(parts.echoes)
    mergeSourceCandidatesPerFieldAndResolve(parts.original)
    updateProjectionLifecycleAndDecisionEvidence()
    preserveRevisionOrderedRemoteEvidenceWithoutInventingCoverage(event)
    persistDependencyWaitsAndDeduplicatedReconciliationIntent()
    receipt = insertAcceptedReceiptAndLinks(event, parts)
    completeManifestEntryIfPresent(manifestEntry, receipt)
    advanceOnlyContiguousCompletedReplayPagesIfEligible()
    COMMIT
    return applied(receipt)
```

Unique-key collisions and deadlocks roll back and retry the entire transaction under the same event identity within the request deadline; exhausted retries return 503. Receipt lookup never bypasses authentication or schema checks. A missing response after commit is safe: the sender retries the same semantic event with a fresh delivery signature and receives committed-duplicate success.

## 4. Responses and invalid events

| HTTP | Meaning and sender action | Accepted business state |
|---|---|---|
| 200 | Applied, applied-blocked, verified echo, or committed duplicate; response includes receipt ID/classification and status. Blocked does not mean synchronized. | Full application transaction committed, or prior identical receipt verified. |
| 400 | Invalid JSON/encoding or duplicate JSON keys. Correct payload. | None. |
| 401 | Missing/invalid signature, unknown key, or expired delivery timestamp. Correct credentials/re-sign delivery. | None. |
| 403 | Authenticated account lacks access or body tenant/account does not match configured authority. | None. |
| 413 / 415 | Oversize body / unsupported content type or compression. | None. |
| 422 | Unsupported schema/object/field/type, invalid create, unsupported full-record semantics, or missing required metadata. Correct contract/payload. | None. |
| 409 | Same event/edit/revision with inconsistent immutable content, source incarnation conflict, mapping collision requiring reconciliation, or unverifiable claimed provenance. Investigate; do not auto-retry changed content under the same ID. | Transaction rolled back; no accepted receipt. |
| 429 | Local admission capacity exhausted before application; bounded Retry-After (provisionally 1 second). Retry same event. | None for that attempt. |
| 503 | Temporary database/lock/dependency-service configuration failure or unknown commit result; Retry-After (provisionally 1 second). Retry same event. | None or already committed if commit outcome was ambiguous; dedup resolves retry. |

Errors return bounded machine reason codes and a request correlation ID without echoing PII. Invalid events may create separate access-controlled, size/rate-bounded `validation_incidents` after rollback; these never constitute acceptance or dedup receipts. If diagnostics cannot be persisted, still reject; do not change to success. Do not keep unauthenticated payload bodies. An authenticated invalid event remains an upstream/client remediation item: repair a decoder/configuration or obtain authoritative corrected history with explicit identity resolution. Never rewrite accepted facts or silently skip a poison replay entry. Invalid manifest entries block the contiguous checkpoint; other independent live events can still be processed. Sender retry behavior and access to recover permanently rejected events remain Q03/Q13 gates.

## 5. Unknown records, dependencies, and full records

An authenticated source create allocates canonical entity and source mapping atomically; concurrent webhook/replay races resolve to one identity through unique constraints. Updates for an unmapped source record establish a PLACEHOLDER with exact edit facts/candidates, preserving whatever fields were actually changed. Do not fill missing required fields with invented values. Request bounded hydration/history work through durable dependency intent; the HTTP handler performs no lookup call. A later create/history event contributes its own original edit and fields under their original revisions. It does not reset fields already edited at newer revisions. A snapshot may supply an identified baseline only with a capture/coverage boundary that prevents it overriding captured edits; it cannot invent missing human history.

For an unknown customer reference, allocate/reuse a typed customer placeholder and source mapping, retain the contact reference as its source ID, and mark contact BLOCKED until the parent is hydrated/active. Unknown owners retain the source principal ID and block the resolved owner projection. Accepted-blocked responses expose incomplete fields and dependency status; stale typed foreign keys are never presented as resolved winners. Parent/principal resolution wakes bounded indexed batches of waiters, each transaction rechecking current winners, so a contact reassigned meanwhile is not attached to an obsolete parent. Remote reference existence does not imply a new remote create is needed. Cross-tenant references are rejected, not resolved by a global ID search. Permanently invalid mappings require visible intervention and do not satisfy 30 seconds.

Version 1 is changed-field-only. A full-record transport needs an explicitly versioned adapter with authoritative changed mask or attributable per-field history. Validate the full snapshot shape and mask; normalize only the fields proved edited into the canonical patch. Unchanged fields never get a new time/actor/revision. If field history identifies multiple original edits, ingest each as its own event through this same routine; do not collapse their identities into one snapshot edit. Without this metadata, return 422 `UNSUPPORTED_CHANGE_SEMANTICS` (or block replay with an incident). Comparing a delayed snapshot to our current row is not a supported substitute.

## 6. Echo, repair, and replay decisions

A verified pure echo only confirms an existing mutation/application at its remote revision. Preserve original actor/time/edit identity; do not advance SALESFORCE human candidates or schedule a return write solely because an echo arrived. It may resolve an uncertain mutation/create mapping when durable unique external-ID/mutation evidence safely establishes identity. A delayed echo cannot mark a newer generation satisfied, lower observed revision, or prove current remote agreement. Follow-up work already needed for newer winners remains durable.

A genuine remote edit that wins updates the portal and normally needs no return patch. A genuine remote edit that loses still enters history; if Salesforce now differs from the resolved winner, durable repair intent is necessary. An older event gives historical evidence only: if no newer trustworthy observation proves agreement, schedule reconciliation rather than assume its historical value is still remote current. Reconciliation may produce zero writes. Even an equal-value edit changes candidate identity/history, but needs no value patch if current agreement is established. Missing provenance is never inferred from actor email; claimed but unverifiable provenance is quarantined through the 409 path.

Replay overlap uses exactly the same event/edit identities, validation, conflict rules, and dedup state. A committed webhook duplicate completes its replay manifest entry atomically without another audit fact or repair. An application crash leaves the checkpoint before incomplete pages; restarting repeats them safely. A later completed page never jumps an incomplete predecessor. Opaque cursor order comes from persisted page links, not numeric comparison. Complete record-history coverage and stream checkpoint coverage are different proofs; neither is inferred from the greatest webhook revision. High-water selection, recovery barriers, shared request accounting, and drain arithmetic remain Phase 6.

## 7. Scenario outcomes and acceptance review

Each row is a manual design trace, not an executed test. S1/S2 are original Salesforce edits; P1 is an existing portal edit. Actor/time are retained on every original fact.

| Scenario | Expected durable outcome and response |
|---|---|
| Identical S1 delivered twice, including concurrently | One fact and field set; one accepted receipt; one set of decisions/intents. Uniqueness/locks serialize effects; both requests return 200 after commit/duplicate verification. |
| Same event ID with changed phone or actor | 409; earlier committed receipt/history remain immutable. No partial new candidate or receipt. |
| Different transport ID aliases the same original edit | New receipt links to existing fact after semantic verification; no second human fact. Different logical identity claiming the same original commit gives 409. |
| S2 phone=C arrives before S1 phone=B, city=Noida | Phone remains S2=C; city advances from S1 where applicable. Both facts audited. Do not discard S1 as an old whole record. 200. |
| Older S1 loses to newer P1 on phone | Audit S1; keep P1 winner. Persist/reuse repair or reconciliation intent if remote agreement is unproved; 200 does not claim remote repair already happened. |
| Distinct S1/S2 both set phone=B | Two human edit facts, latest source candidate S2, no redundant value patch if agreement is established. |
| Failure after fact insert but before receipt/commit | Transaction rolls back facts, candidates, jobs, and mapping changes together. 503; retry applies once. |
| Commit succeeds, HTTP process dies before 200 | Full state/receipt already durable; fresh-signed retry returns duplicate 200 without another effect. Unknown commit outcome follows same rule. |
| Contact update precedes contact create and customer create | Preserve partial contact facts/placeholder, exact parent ID and waiter; 200 applied-blocked. Later original create events hydrate by field revision; wake/materialize latest dependency, preserving newer updates. |
| Unknown owner or unresolved parent | Preserve source values/history and blocked lifecycle. No fabricated mapping or synchronized claim; resolution creates no human edit. |
| Pure echo M for P1 arrives after newer human S2 | Record application evidence for M only; S2 candidates/current remain. No loop; no stale satisfaction of newer work. |
| Mixed event: echo phone plus human city change | Verify phone against M; append one original fact for city only, apply its source candidate, commit both classifications atomically. Unverifiable partition rejects all. |
| Real integration-account change without echo provenance | Process as genuine original edit; do not discard by actor name. Missing required upstream provenance remains an explicit integration risk. |
| Invalid field/null or full snapshot without mask | 422; no accepted receipt/fact/current change. Bounded incident and visible remediation; replay checkpoint cannot skip it. |
| Replay page contains already committed webhook S1 plus new S2 | S1 marks manifest completion without reapplication; S2 applies normally. Checkpoint advances only through fully committed contiguous pages. |
| Clear phone=null, omitted city | Audit phone clear only, preserve city; reject null for required customer reference. |

Conditional healthy inbound latency budget: source commit → authenticated webhook arrival ≤5 s; admission/queue/lock wait ≤10 s; validate/apply/durable commit ≤2 s; portal read visibility ≤3 s. Total `5 + 10 + 2 + 3 = 20 seconds`, leaving 10 seconds against the 30-second requirement. This is an allocation under Phase 1's bounded rates, duplicate/burst load, ≥200 events/s assumed inbound capacity, and resolved dependencies, not a measured worst case. Database reads use committed primary state (or replication lag must fit the visibility bound). Admission retries must fit the same aggregate queue allocation; 429/503 do not reset the source clock or guarantee eventual 30-second compliance. Missing webhooks, dependency blocking, overload, invalid contracts, failures, and catch-up are outside this healthy bound and must be exposed. Production sizing and replay scheduling remain Phase 6.

Validation performed: reviewed each Phase 4 work item and acceptance criterion against the above flow, manually traced duplicates/concurrent receipt races, per-field late edits, mixed provenance, parent hydration, replay overlap, and both sides of commit/response failure. Reviewed compatibility with Phase 2 cases A–N and Phase 3 keys/locks/crash boundaries. No executable tests, production benchmark, live endpoint, or Salesforce capability verification claimed. Acceptance is met as design work under existing upstream assumptions. Remaining gates are Q02/Q13 (payload/auth/retries), Q03/Q04/Q09 (history/replay/coverage), Q10 (per-field provenance), Q11/Q12/Q18 (identity/dependencies/schema), and Q17 (durability/retention). Actual chat export remains pending; no code or commits are created by this phase.
