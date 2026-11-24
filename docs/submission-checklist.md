# Required submission checklist

24 November 2026. Phase 7 is **complete**. After document assembly, the user confirmed that all three outstanding items were taken care of: complete authentic chat history, final repository packaging/access, and revised submission timing/delivery. These external items are closed on user confirmation; the assistant has not independently verified the exports or delivery. No optional implementation has started.

| Item | Evidence now | Remaining action / status |
|---|---|---|
| Design document, all 12 sections | [design.md](design.md), linked specifications and [requirements checklist](requirements-checklist.md) | Written and manually reviewed. Conditional feasibility is explicit; this is not verified production compliance. |
| Open questions, provisional answers and consequences | [assumptions.md](assumptions.md), Q01–Q21; design §1 | Available. Client decisions remain provisional, especially replay/conditional mutation/provenance, timing, quotas and retention. |
| Exactly three primary metrics and escalation values | Design §11 | Available: propagation age, rolling-window request starts, retention safety margin. |
| Ten-minute walkthrough | [walkthrough.md](walkthrough.md), 600-second allocation | Prepared; live delivery/timed rehearsal not performed. |
| Every task chat, every prompt/reply, unedited | Existing [chat_history.md](../Chat_History/chat_history.md); initial inspection recorded below | Resolved by user confirmation. Complete authentic coverage was not independently revalidated by the assistant. |
| Repository with every original commit/message | Local `.git`; initial history inspection recorded below | Final packaging/access resolved by user confirmation. Historical entry-state observations below are not a current repository inventory. |
| Revised deadline and actual delivery | Original brief deadline: 23 November 2026, 10:00 AM IST | Revised timing/delivery resolved by user confirmation. No revised date or delivery receipt was supplied; none is invented here. |
| Optional TypeScript/Node.js ≥20 fake-CRM demo | None | Phase 8 not started; optional code absence is not a required-submission gap. |

## Authentic export process and verification

For eligible ChatGPT accounts, use profile → Settings → Data controls → Export data → Export/Confirm export. Alternatively request Download my data through the [OpenAI Privacy Portal](https://privacy.openai.com/). OpenAI says export availability depends on account/workspace; managed workspaces may require contacting the owner. Exports can take up to seven days; settings download links expire after 24 hours. See the official [export instructions](https://help.openai.com/en/articles/7260999-exporting-your-chatgpt-history-and-data), consulted on 24 November 2026.

These instructions do not establish that this account's export contains every Codex chat. Inspect the actual archive for the planning chat and every phase chat, including Phase 7 and any later follow-ups. If expected Codex chats are missing, use the source application's actual available export/copy facility or contact support; no Codex-specific export control has been verified in this session. Keep the item pending until complete authentic material is supplied.

Preserve original downloaded/source transcript files without trimming prompts, replies, wrong turns or collapsed content. Keep an inventory outside the transcripts with source application, chat title/ID, date range, export method, file path and checksum. Verify first/last messages and continuity against source chats; a hash proves a file stayed unchanged after capture, not that its source was authentic or complete. Include all task conversations and attachments needed to understand them. Keep unrelated account data separate from the task submission; do not rewrite task conversations into a cleaner version. Do not claim this checklist or any summary is the history.

Existing file SHA-256 before Phase 7 edits: `4CDF358E00FC4DC8D1BBAEB6FFE6B550FA7CE8E96CF8F2E838216369FD5823F6`. This is a preservation baseline only. No export was requested on the user's behalf and the existing file was not edited.

## Repository evidence and final packaging

Observed local history at initial Phase 7 assembly, newest first (historical snapshot):

| Commit | Original message |
|---|---|
| `9b6d40411e9286e96cbede4ecc8ec1199c557a20` | Add design specification for outbound synchronization (Phase 5) |
| `87d273d` | Add Phase 3 data model and Phase 4 inbound synchronization specifications |
| `4aa91fb` | Created Implementation Plan |

Configured origin: [bidirectional-crm-sync](https://github.com/Yash-Pratapwar/bidirectional-crm-sync). This records local configuration only; remote contents, access and branch publication were not verified. Three reachable local commits were observed, with no history rewrite.

At Phase 7 entry, `IMPLEMENTATION_PLAN.md` and `docs/assumptions.md` were modified and `docs/recovery-capacity.md` was untracked; those existing Phase 6 changes were reused. Phase 7 adds design, walkthrough and this checklist and updates plan/checklist records. No commit authorization is inferred from phase execution. Once authorized, review all intended changes, use decision-based commits preserving existing history, and provide the repository with history (accessible remote or a verified bundle/full repository), plus any required uncommitted files explicitly identified. Verify the recipient can obtain every intended artifact; a repository URL alone does not prove that.

## End-to-end review record

| Required case | Cross-document invariant reviewed | Outcome / limit |
|---|---|---|
| Duplicate | Receipt/immutable-edit keys, one application transaction; replay duplicate still completes manifest entry | No extra edit or return work; equal-value new commit remains distinct. Design trace, not executed test. |
| Older event | Candidate updates per field/source revision, complete audit | S2 phone survives S1 phone while S1 city can advance. No whole-event revision discard. |
| Loop | Verified provenance separates applications from originals; repairs retain winner identity | Old echo cannot overwrite new candidate; genuine/mixed integration-account edits remain visible. Requires adapter provenance. |
| Six-hour outage | Durable jobs/uncertain identities, global H proof, contiguous checkpoints and fresh per-record history before conditional write | Restart-safe design under upstream/durability contracts. About 24-hour conditional recovery; snapshots do not recover history. |

Checked original 12-section outline, field scope, atomic acknowledgment, actor/time refinement, candidate/tie policy, parent dependencies, stale response protection, strict shared cap and full request/echo accounting. Phase 6 supersedes preliminary 120-minute and nominal latency assumptions. No new contradiction requiring a different business policy was found. Healthy worst-case 30 seconds under bursts remains unproven, not silently declared met. No application, load test or Salesforce capability test was run.

Phase 7 closure: the user subsequently confirmed all three submission gaps were resolved. Earlier export instructions and repository observations document the initial review and remain as guidance/history, not outstanding blockers. Upstream/production questions remain explicitly open as allowed by the brief; they cannot be described as verified guarantees. Phase 8 requires a separate instruction to proceed.
