# Hermes × Paperclip × Mr. Mak — Security Research Plan

Status: Threat model and controls proposal; not a completed audit.
Date: 2026-11-07

## Trust boundaries
1. Paperclip API and its task content: authenticated transport; task descriptions and attachments are untrusted input.
2. Hermes runtime and model provider: execute through existing authorized runtime; do not extract OAuth refresh/access tokens, cookies or session state.
3. Mr. Mak local Node service: loopback/authenticated; avoid exposing new network listeners.
4. Workspace filesystem: bounded allowlisted project roots; no traversal or symlink escape.
5. Git and memory integrations: separate least-privilege identities; sanitize content and secrets.
6. Mobile/Tailscale: existing pairing and permission model must remain enforced.

## Principal risks and mitigations
| Risk | Required control |
| --- | --- |
| Prompt injection in task, research, or repository files | Treat as data, never higher-priority instructions; validate tool calls and policy outside model |
| OAuth token misuse | Use Hermes-supported provider auth internally only; never copy credentials into Mr. Mak, Paperclip, logs or Git |
| Arbitrary command execution | Scoped tool policy, sandboxed working directories, deny dangerous operations by default |
| Path traversal and symlink escape | Canonicalize paths, verify allowed roots after resolution, reject special paths |
| Overwrite of user edits | Expected-revision compare-and-swap, backups, atomic commits, conflict UI |
| Duplicate webhooks and retries | Signed/authenticated event verification where supported, durable idempotency keys |
| Task-status forgery | Paperclip authoritative state; evidence-gated transitions and authorized API calls |
| Agent loops/resource exhaustion | Queue budgets, concurrency caps, timeout, circuit breaker, manual kill switch |
| Secret leakage in artifacts | Redaction, content scanning, no sensitive raw transcripts in cards |
| Supply-chain compromise | Dependency review, lockfile checks, CI tests and no unreviewed installation scripts |
| Overprivileged mobile access | Reuse existing authenticated pairing; no new remote execution route by default |

## Research checklist
- [ ] Inventory service routes, authentication and permissions.
- [ ] Confirm actual Paperclip endpoints, webhook signatures or polling requirements.
- [ ] Confirm Hermes runtime interfaces and OAuth-provider terms/capabilities.
- [ ] Review card schema validator, file update paths and backup restoration.
- [ ] Enumerate all filesystem and shell tools reachable by agents.
- [ ] Define allowlist, role matrix and approval thresholds.
- [ ] Add tests for injection, replay, traversal, symlinks, conflicts, crashes and rollback.
- [ ] Verify telemetry, audit retention and sensitive-data handling.
- [ ] Run isolated integration test with fake credentials and sample tasks.

## Mandatory approval gates
Human approval for deletions, bulk changes, external publication, credential or permission changes, production deploys, financial operations, and high-impact Paperclip state changes. Read-only discovery and local sandbox previews may run unattended within configured budgets.

## Safe implementation order
Document actual interfaces -> write threat tests -> read-only adapter -> guarded workspace writer -> task reconciliation -> opt-in autonomous mode. No live integration or auth changes during research.
