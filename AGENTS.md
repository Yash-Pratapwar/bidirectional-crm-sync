# Tequity assignment: instructions for future Codex chats

Read `IMPLEMENTATION_PLAN.md` before working on this assignment. The original requirements are in `Round1-Tequity.html`; read them when interpreting requirements or resolving discrepancies.

When the user says **"proceed with phase X"**, execute Phase X from `IMPLEMENTATION_PLAN.md`. Inspect its dependencies and existing artifacts first. Reuse completed work; do not restart the project or silently implement later phases. If a required dependency is missing, explain the gap and ask for direction before doing work dependent on that decision. Routine implementation choices within an authorized phase do not require confirmation.

Before changes, give a short update describing the phase and intended artifacts. On completion, update the phase status and execution log in `IMPLEMENTATION_PLAN.md`, record validation and unresolved questions, and report the artifacts produced. Do not mark a phase complete unless its acceptance criteria are met. Preserve decisions and explain changes to assumptions.

The required design, chat history, and repository history take priority over optional code. Code must use TypeScript on Node.js 20+ and fake Salesforce, and run with Node.js alone after `npm install`. Do not introduce a required external Postgres, Redis, Docker, or real Salesforce dependency for the demo. Explain the distinction between demo persistence and production PostgreSQL.

Keep the actual AI chat history unedited. Never fabricate an export or claim a summary is the full chat history. If an authentic export is unavailable, record that submission item as pending. Preserve existing git commits; do not squash or rewrite them. If creating commits is authorized and supported by filesystem permissions, use meaningful decision-based commit messages. Do not create a git commit merely because a phase finished without checking authorization and repository state.

Use the recovery and capacity recommendations in the plan as explicit assumptions, not verified Salesforce capabilities. Do not claim a guarantee that relies on an unavailable upstream capability. If verifying real Salesforce behavior, consult official documentation and record evidence and limitations.

The current request authorizes creating this plan and these instructions only. Implementation phases require a subsequent instruction such as "proceed with phase 1".
