# Android + Termux + Hermes — Mr. Mak Integration Proposal

**Date:** 2026-11-07  
**Status:** Design/research only; no APK, background service, credential or host configuration installed.  
**Branch:** `feature/hermes-paperclip-autonomous-workspace`

## Goal

Extend the current Mr. Mak mobile web app so Android users can see and control Termux-hosted sessions and optionally run Hermes locally, while still seeing Paperclip projects and task status in the same workspace. Preserve the already-working PC-hosted mobile mode.

## Verified baseline

- Mr. Mak already ships a paired Android-compatible mobile web client in `src/mobile/` with a real CLI terminal view, conversations, attachments, read-only workspace reports, and mobile install/add-to-home-screen support.
- `docs/mobile-access.md` confirms mobile uses Tailscale Serve and the **computer** retains running CLI sessions, file data and credentials; a PC must stay awake and reachable.
- The current mobile service is `desktop/service/mobile.mjs`; Android currently acts as a remote client, **not** as a separately integrated Termux runner.
- `desktop/service/agents.mjs` does not list Hermes or Termux as native agent types.
- The existing mobile terminal represents a managed desktop CLI session. A new host identity and transport are needed to represent an Android shell safely.

## Two user journeys

### A. Existing: Android as remote desktop companion (supported now)
Computer runs Mr. Mak Desktop with Codex/Claude/etc; phone uses the paired private mobile link over Tailscale. User can open the Workspace, inspect reports, start supported PC-hosted agents, view terminal output and send input. No Android build is needed. Existing mobile reports are read-only.

### B. New: Android as independent Termux worker (proposed)
Phone runs Termux with its own files, Hermes and other CLI tools. Mr. Mak mobile UI shows an opt-in **Devices / Android Termux** panel; Termux sessions are clearly identified as local Android processes, not desktop PTYs. Paperclip still owns authoritative task status; Hermes reports artifacts through the controlled bridge.

## Recommended design: outbound Termux runner bridge (first implementation)

```text
                   Existing Mr. Mak Desktop service
                 (local authenticated API + event bus)
                              |
              scoped runner pairing + task broker
                              |
                     encrypted private channel
                              |
Android Termux runner  <-----> Mr. Mak mobile PWA (viewer/control)
  - local PTYs                   - Devices panel
  - Hermes CLI                  - Remote terminal
  - Android repository          - Workspace / Paperclip
  - event + artifact client     - Explicit approvals
```

**Transport:** Termux initiates an authenticated outbound connection to an approved desktop/broker address accessible over the user's private Tailscale network. Do **not** expose a public listener on the phone or reuse desktop admin/mobile pairing credentials as runner credentials. A desktop-side broker endpoint is a future implementation and needs an independent authorization and threat review.

**Identity:** distinct stable `runnerId`, `instanceId`, device label, authorized work roots, allowed CLI identities and project memberships. A runner is not an admin and cannot edit arbitrary Paperclip tasks.

**Session transport:** structured session create/attach/input/resize/interrupt/close actions with monotonic sequence IDs, ack/receipt, reconnect recovery, flow control, bounded retained output, UTF-8 sanitation and per-session ownership. Use PTY-compatible local runner implementation on Termux; do not pretend browser JavaScript can execute phone shell commands.

**Task transport:** Paperclip task snapshot is read-only initially, and any Hermes execution requires explicit acceptance by runner owner or a scoped opt-in policy. Deliverables are proposals with source hashes, repo refs and validated paths; merge/publish only through Mr. Mak's guarded card writer.

**Secrets:** Hermes OAuth access stays solely in the existing Hermes auth store in Termux. Paperclip tokens stay where configured and never travel in task text, frontend responses, Android intents, cards or logs. Use short-lived revocable runner pairing credentials, not the desktop bearer token.

**No unsafe default:** runner bridge disabled at start; first implementation only lists running sessions and health; session launch and command input are separately gated.

## Alternative implementation: Native Android wrapper (later)

A Tauri 2 Android app could potentially provide a native `RUN_COMMAND` bridge to Termux. However, the existing desktop build packages a Node service and platform-specific PTY dependencies and is not a drop-in Android APK. A browser-installed PWA cannot directly issue privileged Android `RUN_COMMAND` intents or read Termux output. Native-wrapper work requires Android manifest permissions, explicit user consent, safe fixed command dispatch, Termux `allow-external-apps` opt-in, results handling, and Android-version testing. Treat this as a separate optional track, not an MVP prerequisite.

Termux itself documents `com.termux.permission.RUN_COMMAND` plus `allow-external-apps=true` requirements; enabling these grants potentially powerful cross-app command access and must never be done silently.

## Android MVP interface

- **Devices:** desktop host vs Android Termux host, connectivity, runtime health, device name, permission profile.
- **Terminal:** Termux shell tabs with explicit `Android • Termux` badge, mono text, copy/paste, Ctrl/arrow keys, resize and reconnect.
- **Agents:** Hermes available/absent/version status; user-initiated start, attach, interrupt; display provider name (never tokens).
- **Tasks:** Paperclip issue association, status and project, evidence of execution, no status conflation.
- **Artifacts:** proposal queue, file previews and source links; block cross-device filesystem path assumptions.
- **Security:** paired runner scopes, per-command approval, readonly toggle, remote disconnect/revoke button, audit log.

## Development milestones: Android track

**A0 — Android baseline smoke test** (P0): verify existing PWA on Chrome Android with Desktop+Tailscale; note that reports are read-only; run existing mobile UI tests, capture reconnect behavior and screen layouts. No Termux connection.

**A1 — Termux capability inventory** (P0): inspect device `uname`, Node version, SSH, installed Hermes, PTY, Android process constraints, available free storage; never read OAuth tokens. Record a capability matrix.

**A2 — Simulated runner protocol** (P0): make a mock runner that emits health and terminal output; session authorization, reconnect, event receipts and isolation tests. No shell execution, no phone daemon required.

**A3 — Opt-in Termux runner** (P1): implement local PTY manager with fixed, allowlisted launch targets and bounded working roots; paired outbound link; display real output in the mobile terminal; cancellation and process cleanup.

**A4 — Hermes execution bridge** (P1): use installed Hermes' supported CLI provider/session interface; owner-configured ChatGPT OAuth stays with Hermes; run safe noninteractive test tasks, record evidence.

**A5 — Paperclip + workspace artifact linking** (P1): map Paperclip task to Android runner and proposed card; display status, repo/commit evidence, controlled publishing and rollback.

**A6 — Optional APK exploration** (P2): native Tauri/Android-wrapper feasibility and `RUN_COMMAND` integration, with explicit permissions. No direct mobile app install required to complete earlier milestones.

## Acceptance and safety tests
- Existing mobile pairing remains separate from runner pairing and cannot mint runner/admin identity.
- Deleting a runner pairing immediately blocks new commands and reconnects.
- Session input cannot escape its session; code cannot change an agent's default permissions or bypass mode.
- Android sleep, aggressive background process killing and Tailscale disconnections show accurate stale/offline state.
- Never log OAuth tokens, HTTP bearer values, secret environment variables or raw sensitive transcripts.
- No arbitrary shell command originating from Paperclip text or workspace artifacts.
- User confirms launching a command/session; remote agent is visibly identified as executing on phone.
- Existing desktop and mobile tests keep passing; running services are not restarted without permission.

## References
- Mr. Mak `docs/mobile-access.md`, `desktop/service/mobile.mjs`, `src/mobile/`.
- Termux upstream: https://github.com/termux/termux-app
- Termux `RUN_COMMAND` docs: https://github.com/termux/termux-app/wiki/RUN_COMMAND-Intent
- Tauri 2 Android WebView: https://tauri.app/reference/webview-versions/
