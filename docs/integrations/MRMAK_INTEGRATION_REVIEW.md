# Mr. Mak Integration Review — Hermes + Paperclip

**Date:** 2026-11-07  
**Review type:** Static source review; no deployment or runtime security audit.  
**Scope:** `NewportRiver/mr-mak-workspace` mainline code and public Paperclip/Hermes documentation.

## Verified source findings and recommendations

| Area / source | Verified finding | Recommended change | Priority |
| --- | --- | --- | --- |
| `desktop/service/integrations/manager.mjs` | Existing provider abstraction supports binding and `publish(kind, value)`; bindings persist to local state. | Reuse provider interface but add known-provider registry, strict binding validation, instance/company-scoped identity, replay-safe event queue and tests. | P0 |
| `desktop/service/server.mjs` | Initializes `new IntegrationManager(stateDir)` with no providers. Integration list and bind endpoints exist; artifact import calls `integrations.publish('artifact', ...)`. | Wire a read-only Paperclip provider behind feature flag; add diagnostics, not another listener. | P0 |
| `desktop/service/agents.mjs` | Agents list includes Codex, Claude, Antigravity, OpenCode, Kimi, shell; no Hermes entry. CLI args use agent-specific allowlisted branches. | Add Hermes inventory, safe CLI flags, explicit launch mode; check actual CLI version and parser. | P0 |
| `desktop/service/coordinator.mjs` | Tool schema `open_chat` enumerates supported agents (not Hermes); already has list/read/update workspace and visible agent chat tools. | Add Hermes to validated enum AND all downstream session/UI surfaces; permit only safe scoped actions. | P0 |
| `desktop/service/card-documents.mjs` | Revision-hash checks, serialized writer queue, backups and atomic replacement; registry update can warn after file save. | Reuse as the sole write path for agent changes; emit receipt and add repair/reconciliation for partial registry changes. | P0 |
| `desktop/service/workspace.mjs` | Metadata patch controls active/done/archived, pinned, category with serialized optimistic retries. | Keep display status distinct from Paperclip issue status and introduce explicit mapping rules. | P0 |
| `desktop/service/project-adapters/registry.mjs` | Project capabilities abstraction; defaults to Godot and generic local filesystem. | Add opt-in Paperclip/GitHub project association; don't infer identity from names alone. | P1 |
| `desktop/service/knowledge/providers.mjs` | Local context and Markdown knowledge providers already exist. | Add provenance-aware read-only task/artifact knowledge provider; optional memory index later. | P1 |
| `src/App.tsx` | UI sorts by last touch and automatically archives unpinned non-sample cards older than 7 days, regardless of active status. | Exempt explicitly synchronized active task views or represent live task status in a separate operations pane. | P1 |
| `src/desktop/client.ts` | Desktop bearer token stays in memory; API wrapper sets Authorization; WebSocket authenticates. | Integrate new views through existing client/auth; never deliver Hermes OAuth data to UI. | P0 |
| `SECURITY.md` | Local loopback service; CLI actions have permissions of operating system user; app is NOT OS sandbox. | Sandbox risky execution at OS level; govern provider and project roots; prevent cross-host trust confusion. | P0 |
| `desktop/service/test/workspace-edit.test.mjs` | Tests 401/403, metadata validation/concurrent patches, atomic replacement. | Add adapter, concurrent agent card, event replay, injection, secrets, role and recovery tests. | P0 |

## Immediate design refinements

### 1. Three roles, no competing truths
- **Paperclip:** the task, ownership, stage, approval, priority, source issue IDs, agent execution governance.
- **Hermes:** execution, tool use, structured evidence, artifact and lesson preparation. Reuse its own ChatGPT OAuth-backed model access when configured.
- **Mr. Mak:** visualization, editable reports, project cards and a local audited view of external progress.
- **GitHub:** source/commit provenance. Knowledge stores insights, never the authoritative task state.

### 2. Native integration over screen automation
Use Paperclip's supported authenticated HTTP routes; use Mr. Mak's existing Node service and card editor; launch Hermes via a verified CLI command. Prefer explicit status and artifact payloads over scraping screenshots or PTYs. The agent may use visual tools when visual QA is necessary, but not to simulate API controls.

### 3. Avoid a bidirectional sync loop
Initial one-way flow: Paperclip task → local normalized snapshot → proposed workspace update. After guarded publishing is proven, outbound Paperclip comments/status changes are separate explicit operations, not automatic feedback of all card edits. Origin IDs + operation receipts suppress echo/replay.

### 4. Separate status domains
Paperclip issues may be `todo`, `in_progress`, `in_review`, `blocked`, `done`; Mr. Mak entity status is `active`, `done`, `archived`; Mr. Mak card entry statuses are `idea`, `in-progress`, `review`, `done`. Never use a lossy one-to-one mapping to drive Paperclip transitions. Store an external status snapshot alongside a binding / operations view, not as a fabricated native card status.

### 5. Use existing app auth without credential delegation
The Mr. Mak service issues its own local authenticated bearer token and has a separate mobile pairing boundary. Keep Paperclip credential server-side, scoped to the intended company, and Hermes' ChatGPT OAuth credentials inside Hermes' own auth mechanisms. Do not copy user OAuth state into cards, environment files, logs, task text or UI state. Explicitly enforce permissions before model execution.

### 6. Show meaningful progress instead of false completion
Proposed Operations view:
- Source company / project / issue / assignee
- Paperclip task status and stage; Hermes process state; workspace artifact publication state (separate fields)
- Last confirmed sync, stale/degraded state and correlation trace
- Evidence: commit SHA, test results, review outcomes, artifact previews
- Approve / reject only through a clearly authorized native flow; read-only mobile first

## Prioritized implementation proposals

**Critical first:** known-provider adapter; Hermes CLI support; task identity and binding validation; revision-aware card proposal + crash recovery; audit records; cross-company permission tests.

**Next:** dashboard with durable statuses; artifact manifests and content validation; project mapping; clear origin deep links; knowledge/search provenance.

**Later:** optional memory retrieval, scheduled autonomous reviews, multi-host federation, budget-aware scheduling and full mobile approvals.

## Technical caution and source limitations
- The code review identifies extension points, not an independently proven security vulnerability.
- Server health returns `version: '0.6.0'` in inspected code while root `package.json` is `0.6.3`: a small observability mismatch worth a separate fix/test.
- `IntegrationManager.bind` currently checks `providerId` and `externalId` but accepts other binding keys; tighten validation before adding agent-controlled input.
- Provider interface does not itself implement external polling/webhooks; these must be designed separately.
- `CardDocuments.save` may return a registry-refresh warning after successfully writing card content. External sync must store a recovery step rather than assuming a transaction.
- This review inspected selected source files. Live Paperclip API version and installed Hermes CLI behavior are NOT yet verified.

## Public upstream references
- Hermes CLI: https://hermes-agent.nousresearch.com/docs/reference/cli-commands
- Hermes model providers: https://hermes-agent.nousresearch.com/docs/integrations/providers
- Paperclip API reference: https://github.com/paperclipai/paperclip/blob/master/skills/paperclip/references/api-reference.md
- Mr. Mak root SECURITY.md and desktop/README.md in this repository
