# Phase 1 — Verified Repository Discovery and Integration Contract

Date: 2026-11-07
Status: Source inspection completed for selected Mr. Mak files; Paperclip/Hermes live interface inspection pending. No runtime changes.

## Files examined
- `package.json`: version 0.6.3; React 19, TypeScript 6, Vite 8, Tauri 2; Node test scripts.
- `desktop/README.md`: Tauri webviews, local authenticated Node HTTP/WebSocket service, PTYs, local state in `.mrmak/`.
- `desktop/service/main.mjs`: starts `createService` from `server.mjs`, uses `MRMAK_REPO` and `--repo`, writes non-secret runtime origin to `.mrmak/runtime.json`, native startup pipe handles window credentials.
- `desktop/service/card-schema.mjs`: strict `mrmak-card` version 1; gallery/research templates; item statuses idea/in-progress/review/done; no unknown fields; limits and local image-path validation.
- `docs/editable-cards.md`: file-backed cards, explicit registry entry, backup and revision rules; warns direct agent writes bypass editor concurrency protection.
- `desktop/service/test/workspace-edit.test.mjs`: authenticated `PATCH /api/workspace/entities/:id`, `GET /api/workspace`; 401/403 rejection; concurrent metadata patches preserve independent fields; atomic replacement test.
- `workspace/workspace.json`: `entities` registry with `id`, `title`, `type`, `category`, `folder`, `steps`, `status`, `updated`.

## Architectural implications
1. Implement a service-side integration adapter, not a separate unauthenticated server.
2. Reuse existing authenticated routes only after inspecting `server.mjs` authorization and origin checks in full.
3. Never write card files directly from Hermes in the initial prototype; produce proposals and use a controlled commit path.
4. Separate Paperclip task status from Mr. Mak entity status and item status; mapping requires explicit rules.
5. The UI must not receive Hermes provider credentials. OAuth provider support must be verified inside Hermes.
6. Keep an append-only correlation/audit receipt for each proposed mutation, linked to Paperclip task ID and Git artifact.
7. Do not treat a successful file write as proof that Paperclip task completion was accepted.

## Normalized event contract (proposed, not implemented)
```json
{
  "schema": "nr.paperclip.task-event.v1",
  "eventId": "stable-source-event-id",
  "source": "paperclip",
  "taskId": "task-id",
  "projectId": "project-id",
  "agentId": "agent-id",
  "eventType": "task.updated",
  "occurredAt": "ISO-8601 timestamp",
  "artifactRefs": [],
  "correlationId": "trace-id"
}
```
All fields are validated before use. Task text and artifact content are untrusted. `eventId` must be deduplicated durably.

## Immediate next research tasks
- Inspect `desktop/service/server.mjs` route and authorization implementation, including write paths and origin validation.
- Inspect `desktop/service/workspace.mjs` and `desktop/service/util.mjs` for atomic write, backup and locking semantics.
- Locate exact Paperclip API source/docs and verify supported task lifecycle and authentication.
- Locate exact Hermes runtime source/docs and verify CLI/provider/OAuth and execution interface.
- Produce compatibility matrix: endpoint, method, identity, permissions, payload, retry, side effect, and evidence.
- Add failing security tests before enabling any external event ingestion.

## Decision gate
Proceed to read-only adapter only when Paperclip and Hermes interfaces are verified and Mr. Mak's authentication boundary has been reviewed. Do not introduce live listeners, secrets or task mutations during discovery.
