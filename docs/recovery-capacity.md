# Phase 6: outage recovery and capacity

24 November 2026. Design specification building on Phases 1–5 and the original brief. No executable application, load test, or real Salesforce capability verification is introduced. All upstream interfaces below are required assumptions, not verified Salesforce features.

## 1. Decision and feasibility envelope

Use durable replay, a bounded recovery barrier, and the existing PostgreSQL outbox. Keep every original edit even when its pending field application becomes obsolete. Modified-record snapshots cannot recover overwritten values or actors and cannot satisfy the history requirement.

The provisional 120-minute estimate is replaced. Budget each mapped update as one revision read, one complete record-history page through that revision, and one conditional patch: **three requests per job**, before exceptional overhead. Cached coverage, creates requiring only one call, already-agreed values, and obsolete fields may improve results; the estimate takes no credit for those savings. More history pages, repeated races, or dependency chains increase the cost.

Planning inputs:

| Input | Assumption |
|---|---|
| Original edits | Peak 20/s per direction; quiet 1/s per direction; Phase 1 burst/hot-record bounds retained |
| Outage | 21,600 seconds; PostgreSQL remains durable and portal saves continue |
| Outage history | 432,000 Salesforce edits; no sustained integration echoes while our API calls cannot succeed |
| Boundary uncertainty | At most 1,000 additional replay entries from checkpoint lag, overlap, or pre-outage in-flight applications; measure and replace this input if exceeded |
| Replay page | Up to 500 entries; arithmetic assumes full pages except the last, stable bounded manifest and completion proof |
| Local ingestion | At least 200 delivery/manifest applications/s including duplicates, echoes and database work; reserve 100/s for live deliveries and 100/s for replay |
| Record-history amplification | At most one page per job at the selected revision; revision-associated history available without scanning an unbounded record history |
| Outbound service | Configured pacing 90/s; assume at least 89 useful request starts/s sustained, allowing pacing/transaction overhead; absolute cap remains 100/s |
| Extra requests | At most 10/s including repairs, retries, dependencies, stream pages and control calls; explicit allocation below |
| Replay retention | At least 72 h; mutation/external-ID resolution remains valid for the recovery/retry window |

These throughput floors, page sizes, bounded exceptions and latency limits require adapter confirmation and production measurement. They are conditions for the estimate, not performance guarantees. Account quotas must permit about 7.78 million starts/day at 90/s; a lower daily quota invalidates this model even if the per-second cap permits it.

## 2. What remains on disk during the outage

Portal saves commit authenticated actor/server time, original facts/fields, both-source candidates, decisions, current projections, request-key responses and individual outbound jobs in one PostgreSQL transaction. Existing mappings and unresolved parent/owner waiters remain durable. A portal response means accepted locally, with pending/blocked remote status.

Known no-effect failures retain RETRY and persisted jitter deadlines. Possibly submitted mutations retain immutable request identity/body/precondition, attempt evidence, UNCERTAIN status and their entity dispatch slot. Never discard them because a timeout, restart, or retry count was reached. Cooldown/effective rate, replay cursor/epoch/target, page manifests and per-entry completion remain durable. Uncommitted transactions have no effect.

Use account-wide outage backoff: after repeated connection failures, pause ordinary mutation dispatch and use a single paced health/recovery probe at no more than one per 60 seconds, respecting longer Retry-After. This refines Phase 5's per-job retries to avoid every pending job probing a dead account. Maximum six-hour probe count is approximately 360 plus detection attempts, each budgeted and recorded; bound initial detection attempts by in-flight capacity. A possibly accepted probe mutation still needs ordinary ambiguity handling; prefer a read-only probe. Successful connectivity starts recovery, not blind backlog dispatch.

Salesforce may continue accepting human edits while its API is unreachable to us. Webhooks that arrive still use apply-before-acknowledge; missing ones remain unknown until replay. Never advance checkpoints on elapsed time or apparent webhook quiet. PostgreSQL storage/WAL survival is assumed; database storage loss requires backup recovery outside this process-restart design.

## 3. Recovery epoch, replay completion and safe writes

1. Fence/stop the former egress sender before takeover, wait the Phase 5 full second, restore persisted cooldown and start with no burst credit. Freeze new remote mutations account-wide. Continue portal saves and inbound application. Retain uncertain mutations for reconciliation; read-only discovery may proceed through the shared cap.
2. Persist a recovery epoch and a stable target high-water vector H, one boundary per source partition, from the replay adapter. H must include all committed source events before its capture. Keep the earlier committed cursor C for each partition and an available predecessor boundary for overlap. Never subtract from or numerically compare opaque cursors. If overlap is unavailable, resume exactly from C under the adapter contract; if C has expired, stop with a history gap.
3. Enumerate stable pages from the overlap boundary through H. Persist each page's identity, predecessor/successor, manifest hash and entry identities before application. Require at most two buffered pages per active partition and backpressure fetching; the capacity example uses one partition. Requests for empty/final pages and high-water discovery count too. Repeat a lost fetch from the saved boundary; never infer completion from an empty transient response.
4. Feed every entry to Phase 4's shared transaction. Original facts/actors/times survive, echoes add application evidence, and already-received webhooks complete their manifest entries without another edit. Invalid events or identity/hash mismatches block progress visibly. Independent valid events may proceed, but cannot bridge the checkpoint gap.
5. Under the checkpoint lock, advance C only through linked, fully committed pages, atomically with the final entry/completion evidence. Later pages cannot jump an incomplete predecessor. Mark the epoch's inbound history COMPLETE only after every partition has an authoritative terminal proof through H, all manifest entries through H are committed, and contiguous checkpoints reach H. Persist the proof and completion state transactionally. This means history through H is known, not that both replicas already agree or that no post-H edit exists.
6. Reconcile outstanding uncertain mutations using stable identity, authoritative application/history evidence and the Phase 5 upstream-resolution contract. A request accepted before outage may appear in replay; resolve it without a new human fact. A request still capable of taking effect blocks that entity until resolved/fenced. Do not let a new mutation bypass it. The numerical example assumes these operations fit the bounded overhead and 50-second non-ingestion allowance; long ambiguity makes recovery longer.
7. Release the account mutation barrier after the committed H proof. For every old or fresh job, obtain an observation R and complete history through R, apply that history, then re-evaluate original winner identities under the entity lock. Suppress obsolete unsent fields with explicit outcomes; retain their facts/jobs. Prepare a conditional patch against R only for remaining differences. A source edit after H or after preparation causes history refresh/precondition failure rather than a blind overwrite. A contact still waits for confirmed parent and owner mappings.
8. Continue incremental stream replay after H, capturing successive finite high-water boundaries and advancing contiguous checkpoints. This discovers post-H dropped webhooks, including while backlog drains. Cap stream polling interval at five seconds under healthy service; count empty polls. Global stream coverage does not replace per-record revision coverage unless the adapter supplies a trustworthy mapping proof.

For example, portal P changes phone during outage and unseen remote S changes phone and city. Replay audits S and resolves phone under Phase 2; city survives independently. If S wins phone, P's unsent phone field is superseded. If P wins, its conditional repair follows history coverage. If another remote edit arrives after R, the precondition fails and its history is merged before rebuilding. No recovery-time timestamp is assigned to an old human edit.

A global barrier favors auditable safety and simple coordination; it delays fresh remote writes. Per-partition/record early release is a possible later optimization only with equivalent complete coverage proofs. It is not assumed in these calculations.

## 4. Crash boundaries

| Restart boundary | Durable result and next action |
|---|---|
| Before/after portal commit or webhook commit | Rollback partial work, or recover complete fact/projection/job/receipt; retry by original identity |
| Epoch allocated, H discovery unfinished | Mutation barrier stays closed; finish discovery; do not claim a completion boundary |
| H committed, page fetch lost | Re-fetch from committed cursor/page links to the same H |
| Manifest persisted, some entries committed | Reapply incomplete entries; committed receipts deduplicate overlap; no partial-page checkpoint |
| Later page complete, earlier page incomplete/invalid | Checkpoint remains before gap, even across restart |
| Last entry commits but completion response lost | Read committed markers/proof; repeat safely; never create another fact |
| H proof committed, barrier release not observed | Derive release eligibility from committed epoch/proof; conservatively reconcile before mutation |
| Read/history finishes, mutation preparation not committed | Re-evaluate candidates and observation; no request assumed prepared |
| Mutation prepared, sent, or remotely accepted without local success commit | UNCERTAIN; keep entity slot, resolve exact identity; never blindly reconstruct a different mutation |
| Local success committed, echo arrives later | Verify application evidence; no duplicate original edit or return loop |
| Dispatcher dies or lease expires | Confirm termination/fencing before replacement, wait one second, honor cooldown; lease expiry alone cannot authorize egress |
| Database commit unavailable | Stop acknowledgment/dispatch; no volatile queue substitutes for committed history |

If the process was down as well, extend the replay target/volume to the actual coverage gap. A six-hour API outage plus additional downtime is not a six-hour recovery input.

## 5. Full peak-load arithmetic

### Stage A: recover the missing inbound history

Original outbound jobs: `20 × 21,600 = 432,000`.

Missing inbound entries including bounded overlap: `432,000 + 1,000 = 433,000`. Replay pages: `ceil(433,000 / 500) = 866`. Allow four more discovery/terminal/control calls: **870 calls**. If partitions or API semantics require more, add them explicitly.

Replay can use up to 5 starts/s while ordinary writes are paused: fetch lower bound `870 / 5 / 60 = 2.90 minutes`. Local replay processing is the bottleneck: `433,000 / 100 / 60 = 72.17 minutes`. Fetch and ingest overlap, with buffer backpressure. Add 50 seconds for bounded discovery, first/last-page latency, terminal proof, egress restart pause and initial ambiguity resolution: `(4,330 + 50) / 60 = 73 minutes`. One serial page fetch every five seconds still supplies 100 entries/s with full pages; extra latency/short pages reduce throughput. The 50 seconds is an explicit aggregate assumption, not a consequence of the request cap.

Fresh portal edits accumulated during the barrier: `20 × 4,380 = 87,600`. Jobs when barrier releases: `432,000 + 87,600 = 519,600`. Salesforce also generates 87,600 new original edits during this stage; they are post-H live/incremental-replay traffic, not silently included in the fixed 433,000 entries. With writes paused, live human deliveries plus one duplicate each require at most 40 applications/s. Reserved live capacity of 100/s covers this and bounded outstanding echoes; incremental stream recovery must also be scheduled if webhooks remain missing.

### Stage B: drain outbound jobs while new edits continue

All classes share the sole 90/s paced dispatcher; use a conservative **89/s sustained service floor** in the arithmetic:

| Request class | Starts/s budget | Meaning |
|---|---:|---|
| Ongoing portal jobs | 60 | 20 jobs/s × (read + history page + patch) |
| Conflict/side-effect repairs | 2 | Up to 2/3 complete three-call repair workflows/s |
| Retries/ambiguous resolution | 2 | Additional attempts, status/read/history calls and failed preconditions |
| Mapping/dependency extras | 1 | Extra lookups/parent resolution beyond each edit's baseline |
| Incremental stream history | 1 | Up to 500 entries/s at full pages; includes overlapping echo history |
| Discovery/control | 1 | Boundary/empty polling and operational account reads |
| Exceptional overhead reserve | 3 | Additional pages or transient burst work; do not count it twice |
| Old-job service | 19 | Remaining requests, approximately 6.333 three-call jobs/s |
| Capacity margin below configured pacing | 1 | Allows rounding/persistence/scheduling loss; unused capacity is not burst credit |
| Total | 90 | Every outbound attempt is represented |

Net job drain: `(89 - 60 - 10) / 3 = 19 / 3 = 6.333 jobs/s`.

Drain time: `519,600 × 3 / 19 / 60 = 1,367.37 minutes`.

Total from connectivity restoration: `73 + 1,367.37 = 1,440.37 minutes`, approximately **24 hours**, under these conservative inputs. At an ideal achieved 90/s with no service margin the same model gives `73 + 519,600 × 3 / 20 / 60 = 1,372 minutes` (22 h 52 min). Neither is the earlier 120-minute result.

At the conservative drain rate, baseline mutation attempts are about `20 + 6.333 = 26.333/s`. Add up to 0.667 repair mutations/s: about 27 echoes/s plus 20 human source edits/s = 47 source events/s. With one duplicate per live event, about 94 live applications/s fit the reserved 100/s. Incremental replay at 47 entries/s fits the other 100/s local processing allocation and requires approximately `47 / 500 = 0.094` full page requests/s, plus bounded overlap/empty polls within its 1/s budget. Phase 1's 200/s ingestion assumption therefore remains feasible on paper. More integration side effects, duplicates or short pages require recalculation. Even if post-H webhooks are all dropped, a 100/s replay service exceeds the approximately 47/s source stream, so the post-H backlog can drain under these bounds.

Total Stage B requests at saturation are about `89 × 82,042.11 = 7,301,747`, including about 4.92 million for ongoing portal jobs, 1,558,800 for the fixed old jobs and 820,421 overhead calls. Add Stage A discovery/pages and other actual Stage A calls separately; unused reservations are not fictional requests. In particular Stage A incremental post-H replay, probes, ambiguity resolution and controls are additional actual calls within the shared budget, not part of the 870 fixed-boundary calls.

The dominant bottleneck is outbound request capacity, especially two coverage calls per mutation. Fetching 866 pages is not the dominant cost. Storage and local throughput may become bottlenecks if their assumptions fail.

## 6. Scheduling, latency and sensitivity

Implement weighted fair scheduling of request descriptors, not independent rate buckets. Over recurring service rounds reserve the table's shares, carry incomplete workflow progress durably, select oldest eligible entity within each class, and lend idle shares to other ready classes. Enforce per-record dispatch ordering and parent dependencies. Give live inbound work 100/s local processing and replay 100/s; independent entities progress despite a poisoned replay entry, but completion cannot bypass that entry.

During Stage A, mutation traffic is gated, so only eligible reads/replay/control consume requests. During Stage B, fresh portal workflows receive 60/s and old jobs at least 19/s under the overhead bounds. Repair/ambiguity and dependency classes receive positive service. Bursts use slack or briefly queue; rotate fairly among tenants/entities so a hot record cannot monopolize a class. The table is a sustained-load service model; an implementation must bound short-round scheduling delay and connection usage before claiming a worst-case latency. Any overhead above its allocation reduces old-job service and triggers an updated ETA rather than being discarded.

Catch-up has a separate target. Do not promise fresh traffic within 30 seconds while the global barrier is closed, during backlog drain, or while history polling is recovering gaps. Declare recovery complete only after H coverage, no unresolved pre-outage uncertain mutations or unresolved required historical gaps, old jobs have recorded outcomes, incremental replay is current through a recently captured boundary, dependencies/repairs are resolved, and queues fit healthy latency bounds. Blocked/permanent failures remain visible and prevent a whole-account synchronized claim; an unaffected subset may report its own status explicitly.

Healthy inbound allocation remains `5 delivery + 10 queue/lock + 2 apply/commit + 3 visibility = 20 seconds`. For the explicit three-call outbound workflow, revise Phase 5's two-call example to `2 portal commit + 5 total queue/pacing/wait + 15 sequential API completion + 3 visibility = 25 seconds`, leaving 5 seconds margin. The five-second queue allocation includes waits between all three calls and bounded dependencies. With 20/s healthy fresh work using 60 requests/s and overhead <=10/s, 89/s service leaves 19/s margin. An extra 100 portal edits in a burst need up to 300 requests; the baseline margin alone would take `300 / 19 = 15.79 seconds` to clear them, so these averages **do not prove** the five-second queue bound.

To retain Phase 1's burst envelope and worst-case target, assume healthy per-call completion <=2 seconds (stricter than Phase 5's five-second timeout) and allow aggregate queue/wait <=20 seconds: `2 + 20 + 6 + 2 = 30 seconds`, with destination visibility tightened to <=2 seconds. This is an alternative conditional operating envelope, not an established guarantee; it also requires scheduling/connection tests proving <=20 seconds total wait for every burst workflow, no extra pages/races and resolved dependencies. Otherwise require a smaller admitted burst/stronger combined history interface or disclose that worst-case 30 seconds is unmet. Keep the timeout at five seconds for failure detection, but a call exceeding the healthy bound already invalidates that latency claim. Do not call the sustained throughput calculation a latency proof.

At 89 starts/s and five-second in-flight calls, provision at least `ceil(89 × 5) = 445` concurrent connection/request slots plus operational headroom (e.g. 450); no entity has concurrent unresolved mutations. Database and dispatcher must sustain the corresponding durable accounting. With fewer connections the achieved rate falls; measure it before using this ETA.

For changed inputs, use `T_drain_minutes = a × B / (R - a × p - E) / 60`, only when denominator is positive. Here a=3 calls/job, B=519,600, p=20 new jobs/s, E=10 extra calls/s, R=89. Add barrier time; also recompute B from barrier duration and p.

| Changed sustained condition after the same barrier | Consequence |
|---|---|
| Throttle to R=70 | `70 - 60 - 10 = 0`: no old backlog drain; preserve work, honor cooldown, escalate |
| R=80 | `519,600 × 3 / 10 / 60 = 2,598 min` drain, plus 73 min barrier |
| Portal p=25, fixed initial B for sensitivity only | `89 - 75 - 10 = 4`; drain `519,600 × 3 / 4 / 60 = 6,495 min`; a true six-hour 25/s outage also increases B |
| Two history pages per job, a=4 | `89 - 80 - 10 = -1`: unstable even before draining old work |
| Quiet workload, 1/s each direction | Outage jobs 21,600; history 22,600 including overlap; 46 fixed pages; local replay 226 s plus 50 s = 4.6 min; B=21,876; drain `21,876 × 3 / (89 - 3 - 10) / 60 = 14.39 min`; total about 19 min |

At p=20 and E=10, stability requires `a < (89 - 10) / 20 = 3.95`; at a=3 it requires `p < 26.33 edits/s`. Higher rates, repeated contention, 429 cooldowns, unrecoverable mappings or poison history invalidate the ETA and may prevent convergence. Recompute remaining duration from observed request/job rates; never exceed the cap to meet an old estimate.

## 7. Storage and retention sizing

Illustrative binary units: KiB=1,024 bytes, GiB=1,024^3. Row allowances include associated fields/indexes as stated; they are sizing assumptions to validate with actual schema and distributions, not measured PostgreSQL sizes. Mean two changed fields/edit, <=64 KiB event bound retained; maxima and long values can be much larger than averages.

| Six-hour durable content | Count and assumed allowance | Approximate GiB |
|---|---|---:|
| Both-direction original facts/fields | 864,000 × 1 KiB | 0.824 |
| Decisions and audit lookup indexes | 864,000 × 1 KiB | 0.824 |
| Portal request/outbox/field outcomes | 432,000 × 1.5 KiB | 0.618 |
| Event receipts/application evidence | 864,000 × 0.5 KiB, allowing 432,000 human events plus 432,000 eventual propagation events | 0.412 |
| Mutation bodies and baseline attempt/accounting evidence | 432,000 × 3 KiB, including three request records/job | 1.236 |
| Dependency/replay/control allowance | Fixed provisional reserve, including bounded page buffers/manifests | 0.250 |
| Total incremental live-data envelope | Excludes WAL/replicas/backups and unrelated existing data | **4.164** |

During a complete webhook failure remote facts/receipts are not yet local; that part of the envelope is filled during recovery. Duplicate delivery does not create another original fact or receipt for the same event, but consumes processing and may add bounded diagnostics. Ambiguous attempts and repairs add storage beyond the baseline; budget from actual counts. Up to 360 outage probe records at 1 KiB add less than 0.001 GiB; initial detection calls remain separately counted.

For five million existing entities assume 6 KiB/entity for projections, IDs, candidates/resolutions, dispatch/evidence and indexes: `5,000,000 × 6 / 1,048,576 = 28.61 GiB`. This explicitly includes up to nine business fields per record with two-source metadata; inspect actual row sizes rather than equating record count to history volume.

Peak daily growth is 3,456,000 original edits (40/s), 1,728,000 portal jobs and, during catch-up, up to about 4,060,800 source events/day (47/s), including propagation. Use 2 KiB/original for fact+decisions, 4.5 KiB/portal job for outbox/request+mutation/three attempts, and 0.5 KiB/source event for receipt/evidence. This gives `6.592 + 7.416 + 1.936 = 15.944 GiB/day`, before extra requests. The full 10/s overhead at 1 KiB additional attempt/evidence per request adds `864,000 / 1,048,576 = 0.824 GiB/day`: approximately **16.77 GiB/day**. Event buffers/control reserve and long fields need separate headroom. This is conservative for steady state but may underestimate unusually large payloads.

For six hours plus approximately 24-hour recovery (30 hours), incremental live data is about `16.77 × 30 / 24 = 20.96 GiB`. Example initial storage envelope: `(28.61 + 20.96 + 0.25) × 2 = 99.64 GiB`, allowing a factor of two for bloat/free working space, **plus** separately provisioned WAL/temporary storage (illustratively another 50 GiB, to measure), plus historical data already retained, backups and replicas. This is a roughly 150-GiB initial primary-disk planning envelope, not a fixed production size or a storage guarantee. Every additional retained peak day adds approximately 16.77 GiB logical data before headroom. Never prune required history merely to fit the example disk.

72-hour upstream replay retention covers a six-hour outage plus about 24-hour recovery, leaving about 42 hours margin. Retention must apply to the oldest uncommitted cursor and every required partition, including records blocked by poison entries. Trigger intervention when `remaining upstream retention < estimated remaining recovery + 6 hours safety`, or immediately on a detected gap/no positive drain. This recovery-specific guard does not select Phase 7's exactly three primary operational metrics. Persist dedup/mutation identities for their full retry/replay horizon; local audit retention is a separate client policy and is not 72 hours. If upstream history expires, snapshot repair can restore limited current state, but mark missing historical edits/actors unrecovered and require archive/integration access or an explicit requirement change.

## 8. Acceptance review and open decisions

Manual review traces the restart table, concurrent remote edit example, contiguous replay proof, parent/uncertain-write gates, rate allocation, echo amplification and storage growth against Phases 2–5. Arithmetic is checked separately; no production throughput or executable recovery tests are claimed. The full request model admits eventual drain under the stated bounded workload and upstream contracts. Snapshot-only history recovery is explicitly rejected. The 120-minute estimate is superseded by about 24 hours at the conservative service floor.

Remaining client/production gates: Q03/Q04 stable complete replay, page sizes/partition completion and retention; Q08/Q09 safe create, bounded revision-associated history and outstanding mutation resolution; Q10 attributable echoes/side effects; Q11/Q12 dependency resolution; Q14 achieved throughput, quotas, network cap semantics and amplification; Q15 catch-up latency policy; Q17 measured storage/durability and audit retention. The original 30-second worst-case target under the allowed burst remains a feasibility gate requiring the stricter healthy envelope and measured scheduling bounds above, or an agreed contract/workload change. This limitation is recorded rather than presented as verified compliance. Phase 7 must carry it into the consolidated design/open questions. Authentic full chat export remains pending; existing chat files and git commits are untouched. Phases 7–8 are not implemented here.
