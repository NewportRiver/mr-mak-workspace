# FEATURE PLAN — Hermes × Paperclip × Mr. Mak + Android Termux

**Development branch:** `feature/hermes-paperclip-autonomous-workspace`  
**Updated:** 2026-11-07  
**Status:** Design and source research complete for selected components; no integration code implemented yet.  

## The idea

Use Mr. Mak as the visual workspace for work coordinated in Paperclip and executed by Hermes through its independently configured model providers (including ChatGPT OAuth-backed Codex where supported). Add an opt-in Android/Termux runner so Hermes and local terminal sessions can operate on the phone, with results and approvals visible alongside desktop-hosted agents.

**System responsibilities**
- **Paperclip:** tasks, ownership, priorities, issue status and approvals.
- **Hermes:** AI agent execution, tools, memory and verification; credentials stay inside Hermes.
- **Mr. Mak:** project and deliverable cards, audit views, terminal UI and work evidence.
- **Termux:** optional independently paired Android process host; no browser-to-shell shortcut or token reuse.

## Development milestones

| Milestone | Feature | Current status |
| --- | --- | --- |
| M0 | Source and API compatibility research | Mr. Mak source inspected; deployed Hermes/Paperclip versions pending |
| M1 | Paperclip read-only task/provider integration | Planned |
| M2 | Link Paperclip tasks to existing Workspace cards | Planned |
| M3 | Add Hermes as selectable local CLI agent in Chats | Planned |
| M4 | Guarded artifact publishing, approvals, backups and audit | Planned |
| M5 | Agent Operations dashboard, task/evidence/health view | Planned |
| M6 | Project knowledge and provenance-backed memory | Planned |
| M7 | Bounded autonomous test pilot | Planned |
| M8 | Multiple host/remote worker federation | Future |
| A0–A6 | Existing Android PWA tests through Termux runner MVP and optional APK exploration | Planned |

## Detailed plans and research

Everything is in [docs/integrations/](docs/integrations/):

1. [DEVELOPMENT_MILESTONES.md](docs/integrations/DEVELOPMENT_MILESTONES.md) — M0–M8 milestone tasks, dependencies, success criteria, tests.
2. [MRMAK_INTEGRATION_REVIEW.md](docs/integrations/MRMAK_INTEGRATION_REVIEW.md) — source review and prioritized improvements to existing integration manager, card writer, CLI and UI.
3. [ANDROID_TERMUX_INTEGRATION_PROPOSAL.md](docs/integrations/ANDROID_TERMUX_INTEGRATION_PROPOSAL.md) — Android PWA status, Termux runner bridge and A0–A6 milestones.
4. [HERMES_PAPERCLIP_ARCHITECTURE.md](docs/integrations/HERMES_PAPERCLIP_ARCHITECTURE.md) — proposed architecture, event flows and phased gates.
5. [HERMES_PAPERCLIP_SECURITY.md](docs/integrations/HERMES_PAPERCLIP_SECURITY.md) — threat model, auth and approval boundaries.
6. [COMPATIBILITY_MATRIX.md](docs/integrations/COMPATIBILITY_MATRIX.md) — source-verified interfaces and unresolved runtime checks.
7. [PHASE1_REPOSITORY_DISCOVERY.md](docs/integrations/PHASE1_REPOSITORY_DISCOVERY.md) — selected existing Mr. Mak APIs, code files and test coverage.
8. [UPSTREAM_CONTRIBUTION_PLAN.md](docs/integrations/UPSTREAM_CONTRIBUTION_PLAN.md) — contribution route to `witnesstodark/mr-mak-workspace`.

## Immediate next deliverable

Validate the deployed Paperclip OpenAPI routes and installed Hermes CLI flags/authorization behavior; then implement a **read-only Paperclip provider** and **Hermes selectable chat agent** as individually testable, opt-in additions. No production write automation until revision, authorization, approval and replay tests pass.

## How to contribute upstream

This branch lives in the `NewportRiver` fork; it does **not** appear in the original repository's branch selector. Once implementation has isolated tests, propose a focused pull request from the fork to `witnesstodark/mr-mak-workspace:main`. The current branch has research/design files, not working feature code.
