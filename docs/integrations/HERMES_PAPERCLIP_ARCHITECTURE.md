# Hermes × Paperclip × Mr. Mak — Architecture Research Plan

Status: Research/design only; no runtime integration enabled.
Branch: `feature/hermes-paperclip-autonomous-workspace`
Date: 2026-11-07

## Mission
Use Paperclip as the authoritative task/delegation system, Hermes as the agent execution and verification coordinator, and Mr. Mak as the human-facing workspace and deliverable catalog. Preserve upstream functionality and provide a reversible opt-in integration.

## Verified Mr. Mak baseline
- v0.6.3: React 19, TypeScript, Vite, Tauri 2, Node service.
- `src/` renders workspace and chat views; `desktop/service/` provides local HTTP/WebSocket, PTYs, history and tools.
- `workspace/workspace.json` indexes cards; `content.card.json` contains editable research/gallery content.
- Cards have schema validation, atomic-write guidance, local backups, revision conflict checks in desktop editing.
- Service binds to loopback and uses authenticated local URLs. Mobile access is separately paired via Tailscale.
These are repository-documentation findings, not runtime verification.

## Proposed data flow
1. Paperclip task event or bounded polling -> adapter (authenticated, read-only initially).
2. Adapter maps task, project, agent, status and deliverable references to normalized event envelope.
3. Hermes controller receives scoped work order, plans and executes with its configured provider, tools and memory.
4. Verification stage validates artifacts, task provenance, hashes and completion criteria.
5. Workspace adapter prepares a proposed card/registry update and runs schema + conflict validation.
6. Approved update is atomically committed; receipt and idempotency key are recorded.
7. Paperclip is updated only via authorized task-state API, with correlation IDs; failures are retried safely.

## System ownership
- Paperclip owns tasks, assignments, approval state, execution priority and task status.
- Hermes owns execution sessions, tool use, reasoning and evidence gathering.
- Mr. Mak owns workspace presentation, card metadata and indexed deliverables.
- Git owns source version history; memory stores retrievable knowledge with provenance, not task truth.

## Interface research (no assumptions)
- Inspect Paperclip API schema, webhook availability, authentication, task transitions, agent identity and rate limits.
- Inspect Hermes runtime adapter/CLI, OAuth-backed provider capability and authorization boundaries; do not export or reuse login tokens.
- Inspect Mr. Mak service routes, card schema, registry update path, revision/backups, existing permissions and PTY launch behavior.
- Confirm each supported capability through code and tests before implementation.

## Proposed modules (not yet implemented)
- `desktop/service/integrations/paperclip/`: event intake, normalization, status sync.
- `desktop/service/integrations/hermes/`: scoped execution gateway, session tracking.
- `desktop/service/integrations/workspace/`: card proposals, validation and atomic commit.
- `desktop/service/integrations/audit/`: append-only receipts, deduplication and replay.
- `src/`: optional Agent Operations view with pending approvals, task links and execution evidence.

## Delivery phases and gates
### A — Discovery and threat modeling
Read code and integration docs, enumerate interfaces, trust boundaries, supported OAuth flows, and existing tests. Deliver compatibility matrix and threat model. No credentials or writes.
### B — Read-only prototype
Feature flag off by default. Import sample Paperclip task metadata, render proposed cards in a sandbox, and verify task-to-artifact provenance. No task mutation.
### C — Guarded writes
Explicit workspace allowlist; validation, expected revision/compare-and-swap, atomic writes, backups, dry-run, audit receipts, rollback and user approval for destructive changes.
### D — Autonomous orchestration
Bounded queues, concurrency/resource budgets, retries with idempotency, timeouts, human approval checkpoints and safe Paperclip status reconciliation.
### E — Dashboard and pilot
Agent activity, evidence, failed jobs, approvals, sync drift, latency and rollback UI. Pilot one noncritical project before broader rollout.

## Acceptance criteria
- Original Mr. Mak tests and desktop/mobile workflows remain functional.
- No provider credentials exposed to browser, logs, Git, Paperclip task text or workspace cards.
- Duplicate/replayed events do not duplicate cards or mutate tasks twice.
- Simultaneous user/agent edits are detected and preserved.
- Failed jobs cannot falsely mark Paperclip tasks complete.
- Disable flag stops integration without impacting existing workspace.
- Every mutation has an actor, task ID, timestamp, before/after hash and recovery path.
- Integration remains non-destructive until separately approved.
