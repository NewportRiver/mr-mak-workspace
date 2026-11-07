# Autonomous Workspace — Development Milestones

**Initiative:** Mr. Mak × Hermes (ChatGPT OAuth-backed) × Paperclip  
**Branch:** `feature/hermes-paperclip-autonomous-workspace`  
**Updated:** 2026-11-07  
**Status:** Research and planning; implementation not started. Do not enable live synchronization during this phase.

## End state
Paperclip remains authoritative for task assignment, approvals and lifecycle. Hermes is a bounded execution and verification worker using its OWN already-authorized model/provider session (including OpenAI Codex via ChatGPT OAuth when configured). Mr. Mak becomes a visual catalog of projects, deliverables and evidence, with a governed agent-operations view. No tokens are copied across products.

## Project rules
- Keep `main` unchanged; all work on this feature branch with individually reviewable commits.
- Work offline/read-only first; opt-in live connections must be disabled by default.
- Existing Tauri desktop, chats, card editing, history, mobile pairing and local credential protections must continue working.
- Keep operational queues, cursors, receipts, and secrets out of Git under `.mrmak/` or a suitably protected system store.
- Treat remote task instructions, repository content and artifacts as untrusted data, never an authorization source.
- Avoid a second task database: Paperclip owns task truth. Avoid a second card writer: reuse `CardDocuments` with an authorization-aware integration boundary.
- Do not restart applications, touch real credentials, change Paperclip configuration or launch agents without explicit approval.

## M0 — Compatibility and baseline inspection (P0)
**Objective:** Establish exact, tested integration contracts before implementation.
**Targets:** `desktop/service/server.mjs`, `agents.mjs`, `sessions.mjs`, `coordinator.mjs`, `card-documents.mjs`, `integrations/manager.mjs`, `project-adapters/`, `src/desktop/`.
**Tasks**
- Inventory Mr. Mak service endpoints, client auth, mobile boundaries, event broadcasts, card editor and audit paths.
- Verify installed/versioned Hermes CLI capabilities (interactive session, `--query-file`, `--oneshot`, resume, provider selection) on the intended host, without reading credentials.
- Verify deployed Paperclip `GET /api/openapi.json` and documented issue, agent, project, comment and attachment routes with read-only requests.
- Record connectivity and per-host boundaries (Windows/WSL/remote instances), allowed roots and project identity matching; never assume that all systems share a filesystem.
- Record baseline tests: `npm run test`, `npm run build`, `npm run lint` and existing UI test commands where supported.
**Deliverable:** `COMPATIBILITY_MATRIX.md` with verified methods, versions, response shape, auth principal, source and unknowns.
**Done when:** Exact interfaces and a threat-model review are documented, baseline tests recorded, no production state changed.

## M1 — Paperclip read-only provider (P0; depends M0)
**Objective:** Import task/project/agent summaries into a local in-memory/sandbox model.
**Tasks**
- Extend existing `IntegrationManager` with a configured `paperclip` provider instead of duplicating it.
- Implement explicit opt-in, allowlisted Paperclip server URL, narrowly scoped credential storage, timeouts, pagination, backoff and connection health.
- Fetch company/project issue listings and individual issue details; normalize immutable task snapshots.
- Record source task ID, source company and instance ID, project ID, assignee, status, source update cursor, timestamp and provenance.
- Start with manual refresh/polling. Webhooks only if their authentication/replay semantics are confirmed on the deployed Paperclip version.
- Add transport mocks and fixtures; forbid live task mutations.
**Done when:** Offline tests show complete, deduplicated, redactable task summaries; disconnected startup and bad credentials fail safely.

## M2 — Workspace bindings and safe proposals (P0; depends M1)
**Objective:** Link Paperclip work to EXISTING Mr. Mak cards without duplicating or clobbering edits.
**Tasks**
- Tighten `IntegrationManager.bind` schema: known provider, source-instance+company+issue identity, existing workspace entity ID, no arbitrary keys.
- Introduce a durable correlation table keyed by source instance/company/issue + workspace entity ID; avoid ambiguous title matching.
- Implement a read-only proposed diff and `GET` preview route, with artifact manifest and expected card revision.
- Reuse existing `CardDocuments.read/save/create` paths for any eventual writes; require expected revision, limit/path validation, backup and atomic registry update.
- Refuse symlinks, traversal, unknown card schema versions, stale revisions and ambiguous bindings.
**Done when:** Replayed events are no-ops, user edits are preserved and cross-project data cannot be linked accidentally; no automatic writes.

## M3 — Hermes native runner integration (P0; depends M0 and threat tests)
**Objective:** Make Hermes a supported, visible agent in Mr. Mak.
**Tasks**
- Add `hermes` to agent discovery/inventory; implement tested terminal command construction for the ACTUAL installed version, not guessed flags.
- Support visible interactive PTY sessions; separately implement one-shot headless jobs only through bounded worker queue.
- For noninteractive task text, use Hermes' documented `hermes chat --query-file ... --oneshot` where supported; never concatenate an issue description into shell commands.
- Preserve Hermes' private ChatGPT OAuth/Codex auth store; never import access/refresh tokens into Mr. Mak settings, Paperclip tasks, workspace data, logs or browser state.
- Extend session lifecycle, conversation parsing, history, resume (only after verified), mobile display, UI picker, permission gates and tests; never silently enable bypass.
- Prevent an agent replying from being mistaken for a completed task.
**Done when:** A user can deliberately open Hermes as a visible worker using an existing local sign-in, cancel it, and review session evidence with no leaked credentials.

## M4 — Guarded publishing and reconciliation (P0; depends M1–M3)
**Objective:** Publish verified deliverables with a full audit trail.
**Tasks**
- Persist operation intent, stable idempotency key, source snapshot, before/after hashes, proposed file paths and actor.
- Review work against acceptance criteria before publishing; validate MIME, sizes, content and path roots; only copy allowlisted artifacts.
- Apply writes through the guarded card service after revision check and approval policy; store atomic backup receipts; handle partial registry update failures.
- Add reconciler: check Paperclip authoritative status before any outbound status/comment action; require authorized Paperclip identity and native review gates.
- Do not mark done on transport success alone; require evidence and returned authoritative state.
**Done when:** Partial failures, retry/replay, conflict, interrupted run and crash-recovery tests pass; every side effect is traceable and reversible.

## M5 — Agent Operations dashboard (P1; depends M1–M2)
**Objective:** Display task cards, assigned agents, evidence, queue/health and approvals without replacing Paperclip.
**Tasks**
- Add scoped views: Inbox, Working, In Review, Blocked, Completed, Failed Sync, Pending Approval.
- Deep link to Paperclip issue and GitHub commit/artifact; show source attribution, last sync and per-host runner status.
- Resolve dashboard vs Workspace card display semantics: `status: active|done|archived` is not Paperclip issue status.
- Avoid seven-day auto-archive hiding ongoing synchronized work; preserve existing behavior for ordinary cards.
- Make mobile display read-only before mobile approvals or remote agent controls are considered.
**Done when:** Users can distinguish 'not synced', 'agent executing', 'artifact proposed', and 'verified complete' at a glance.

## M6 — Knowledge and project mapping (P1; depends M2–M4)
**Objective:** Make project memory reusable, attributable and non-authoritative.
**Tasks**
- Add Paperclip/GitHub-aware project adapter in `project-adapters/` and a knowledge provider in `knowledge/providers.mjs`.
- Extract approved task summaries, decisions, test results and lessons with provenance, retention and content limits.
- Optional SQLite indexing and external memory bridges (e.g. Hermes memory) only behind independent feature flags.
- Maintain source-of-truth links; memory must never override Paperclip status or permissions.
**Done when:** Results are source-cited, project-scoped, revocable and robust to deleted/reassigned tasks.

## M7 — Autonomous pilot and operations (P1; depends M4–M6)
**Objective:** Enable bounded autonomy on ONE noncritical test project.
**Tasks**
- Set conservative per-host concurrency (initial default one active Hermes job), budget, heartbeat/wake schedule, queue limits and kill switch.
- Implement exponential retry/backoff, circuit breaker, dead-letter handling, sync drift detection and manual recovery.
- Use Paperclip's native issue checkout, ownership, execution policy and approval gates when acting as a Paperclip agent.
- Pilot at least one complete lifecycle: assign → execute → validate → propose → approve → publish → reconcile.
- Document rollback and metrics: completed tasks with evidence, duplicate-event rate, merge conflicts, stale status, average queue latency.
**Done when:** Pilot passes fault-injection and approval tests; no unrelated files or tasks are touched.

## M8 — Multi-host federation (P2; depends M7)
**Objective:** Support separate Paperclip/runner hosts without assuming shared identities, filesystem or credentials.
**Tasks:** Per-instance IDs and auth; explicit remote artifact transfer; host health and worker ownership; deploy sequencing and governance. No implicit SSH, filesystem mounts or token forwarding.
**Done when:** A disconnected subsidiary host cannot corrupt the parent workspace and reconnects without duplicate execution.

## Suggested implementation order
M0 → M1 → M2; M3 may proceed after M0 and security tests; then M4 → M5/M6 → M7 → M8.

## Minimum acceptance test suite
- Unit: normalized issue schema; task-to-card binding validation; status mapping; artifact path validation.
- Adversarial: prompt injection, hostile filenames, symlinks, traversal, invalid URL/host, forged webhooks, unexpected API response, secret redaction.
- Consistency: replay, duplicate event, out-of-order updates, user-vs-agent conflict, interrupted atomic write, partial registry failure.
- Auth: no unauthorized API access, cross-company isolation, no elevation via coordinator, mobile cannot gain new write permissions.
- Runtime: no silent CLI flag changes, cancellation, subprocess cleanup, stdout log sanitation, resource-budget and queue enforcement.
- Regression: existing desktop tests, UI tests, build, lint and mobile pairing.
