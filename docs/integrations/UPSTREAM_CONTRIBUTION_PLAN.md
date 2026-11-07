# Upstream Contribution Strategy — Hermes, Paperclip and Android Termux

Date: 2026-11-07
Status: Proposed; no upstream pull request submitted.

## Repository relationship
- Original upstream: `witnesstodark/mr-mak-workspace`.
- Contributor fork: `NewportRiver/mr-mak-workspace`.
- Working branch in fork: `feature/hermes-paperclip-autonomous-workspace`, created from `main`.

The fork is the correct place to develop and test changes. Upstream contributions are proposed via cross-repository pull requests from the fork's feature branch (or preferably from smaller focused branches) to `witnesstodark/mr-mak-workspace:main`. Do not merge experimental work into the fork's main merely to submit it upstream.

## Distinguish proposal from implementation
This working branch currently contains design/research documentation. It does NOT contain a functional Hermes/Paperclip adapter, Android native app, Termux runner, or verified integration tests. Any PR must say so plainly.

## Recommended small reviewable PRs
1. **Proposal/issue:** Ask upstream whether optional Hermes CLI support and a Termux runner match their scope, plus the preferred integration point. Link concise architecture notes rather than requiring upstream to review every private-company operations plan.
2. **PR A — Hermes selectable CLI:** standalone, optional adapter for `hermes` in inventory, sessions, UI and test suite. No Paperclip dependency, no OAuth token copying.
3. **PR B — Paperclip provider:** read-only project/task data provider using existing `IntegrationManager`, explicit opt-in and tests. No automatic task writes.
4. **PR C — Android Termux sessions:** extend existing mobile PWA with a paired, least-privilege runner bridge; separate session identity and careful disconnect handling. No assumption that the browser can directly run Termux.
5. **PR D — Operations and publishing:** guarded task-to-card binding and audit trail only after upstream accepts basic provider semantics.

## Proposed cross-fork workflow
1. Keep upstream `main` as the integration baseline; fetch upstream changes periodically.
2. Implement and test each feature on a focused fork branch (e.g. `feature/mrmak-hermes-cli`, `feature/mrmak-termux-bridge`) based on upstream `main`.
3. Run `npm test`, `npm run build`, `npm run lint`, and applicable desktop/mobile UI tests before submission.
4. Verify no `.env`, auth state, sensitive logs, CLI histories, personal workspace documents or generated large media in diffs.
5. Open a DRAFT cross-fork PR with summary, why upstream users benefit, screenshots from real tests, test receipts, limitations and privacy/permission details.
6. Obtain maintainer feedback, revise, and only then mark ready for review.
7. Keep NewportRiver-specific multi-host federation and Paperclip company workflows as optional extensions that do not burden the upstream default installation.

## Contributor's rationale
The generic upstream value is *supporting another CLI provider* and *optionally attaching a local/remote Android worker*. A NewportRiver corporation hierarchy is an optional example, not a requirement imposed on all users.

## Suggested upstream proposal summary
> Extend Mr. Mak's existing CLI and phone architecture with two opt-in provider adapters: Hermes as a local CLI agent, and an Android Termux runner connected through a distinct, scoped bridge. This would allow users to keep the familiar Workspace/Chats UI while choosing whether execution occurs on their desktop or Android device. A separate Paperclip provider can surface task/project context without taking over existing source-of-truth or permissions. Initial implementation would be read-only and preserve existing mobile pairing, sandboxing and revision checks.

## Caution
A branch in the fork is not automatically visible in the original repository's list of branches. Upstream will see it when a pull request is opened or a link is shared. No upstream action has been requested on their behalf.
