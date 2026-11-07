# Integration Compatibility Matrix (Discovery Draft)

**Date:** 2026-11-07  
**Evidence level:** Mr. Mak source and public upstream docs checked; deployed Paperclip and installed Hermes runtime remain unverified.  
**Use:** Update with real versions/test receipts before enabling M1 live access.

| Capability | Integration surface | Source evidence | Read/write | Auth boundary | Verification / blocker |
| --- | --- | --- | --- | --- | --- |
| List workspace | `GET /api/workspace` | Mr. Mak `desktop/service/server.mjs` | Read | Mr. Mak local bearer + host/origin | Local test observed in repo; runtime pending |
| Read card content | `GET /api/workspace/cards/:id/:step` | Mr. Mak `server.mjs` + `card-documents.mjs` | Read | Mr. Mak local bearer | Schema/revision returns; no external actor auth yet |
| Save card | `PUT /api/workspace/cards/:id/:step` with revision | Mr. Mak `server.mjs` + `card-documents.mjs` | Write | Mr. Mak local bearer | Existing guarded write; require integration policy + explicit approval |
| Create card | `POST /api/workspace/cards` | Mr. Mak `server.mjs` | Write | Mr. Mak local bearer | Ensure duplicate detection and binding before automation |
| Edit metadata | `PATCH /api/workspace/entities/:id` | Mr. Mak `server.mjs` + `workspace.mjs` | Write | Mr. Mak local bearer | Different state domain from Paperclip task |
| Existing provider bindings | `GET /api/integrations`, `POST /api/integrations/bind` | Mr. Mak `server.mjs` + `integrations/manager.mjs` | Read/write | Mr. Mak local bearer | No provider configured; strict schema and scoped identity needed |
| Paperclip agent identity | `GET /api/agents/me` | Public Paperclip skill/API reference | Read | Paperclip-owned credential | Compare deployed OpenAPI and permitted roles |
| Paperclip issue list | `GET /api/companies/:companyId/issues` | Public Paperclip API reference | Read | Company/agent permissions | Verify deployed query schema/pagination |
| Paperclip issue detail | `GET /api/issues/:issueId` | Public Paperclip API reference | Read | Paperclip | Verify deployed response and status semantics |
| Paperclip issue comments | `GET /api/issues/:issueId/comments` | Public Paperclip API reference | Read | Paperclip | Confirm cursors/pagination and access |
| Paperclip task updates | `PATCH /api/issues/:issueId` | Public Paperclip API reference | Write | Paperclip authorized actor + execution policy | Defer until M4; do not bypass review gates |
| Paperclip issue checkout | `POST /api/issues/:issueId/checkout` | Public Paperclip API reference | Write | Paperclip authorized actor | Defer until M7; enforce source ownership |
| Paperclip OpenAPI | `GET /api/openapi.json` | Public Paperclip API reference | Read | Deployed instance | Required first live API compatibility check |
| Hermes configured model | `hermes model`, `hermes auth` | Official Hermes docs | Credential management | Hermes ONLY | No token export/import; installed version pending |
| Hermes interactive CLI | `hermes chat` | Official Hermes CLI reference | Execute | Local logged-in Hermes user | Check installed binary, PTY/shell behavior |
| Hermes one-shot | `hermes chat --query-file PATH --oneshot` | Official Hermes CLI reference | Execute | Local logged-in Hermes user | Verify installed flags; bound work queue, no untrusted shell quoting |
| Hermes ChatGPT OAuth | OpenAI Codex provider through Hermes `hermes model` | Official Hermes provider docs | Inference | Hermes own auth store | Do not treat as generic OpenAI API key or Paperclip credential |

## Planned normalized read-only DTO

```json
{
  "schema": "nr.paperclip.snapshot.v1",
  "source": { "kind": "paperclip", "instanceId": "instance-key", "companyId": "company-id" },
  "issue": { "id": "issue-id", "projectId": "project-id", "assigneeAgentId": null, "status": "todo", "updatedAt": null },
  "collectedAt": "2026-11-07T00:00:00.000Z",
  "provenance": { "requestId": "local-trace-id", "readOnly": true }
}
```

Do not use this draft as a live request/response schema. Verify `/api/openapi.json` before coding.

## Open questions to resolve in M0
1. Which exact Paperclip version and local/remote base URL is deployed? Is its OpenAPI current?
2. What principal and permissions can list assigned tasks without status update capabilities?
3. Which Hermes version is installed and on which hosts? Is `--query-file` supported and safe for service-run use?
4. How does the target Hermes install resume interactive sessions and expose task completion evidence?
5. Can Paperclip's native execution stage and approval policies be surfaced without copying or bypassing auth?
6. What approved artifact roots and cross-host transfers are allowed?
7. What is the minimum acceptable retention, encryption and redaction policy for audit records and memory?
