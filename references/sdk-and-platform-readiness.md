# SDK And Platform Readiness

Use this reference to keep game integration separate from SDK packaging and Funloom platform capability work. Lessons from one test game should become reusable rules, not one-off game patches.

## SDK Packaging State

Public creator onboarding should use the official npm distribution documented in `docs/funloom-sdk`:

- `@funloom/sdk` for server, browser, and webhook helpers
- `@funloom/cli` for the `funloom` command

The current public docs list `@funloom/sdk@0.1.1` and `@funloom/cli@0.1.1`. If newer SDK docs are supplied, use those docs as the source of truth.

Record:

- package source and version actually installed
- install command actually used
- lockfile changes
- for any explicit internal fork only: repo source plus tag or commit

Do not write public instructions that depend on internal absolute paths, test branches, or unpublished package names. Use a local path, Git dependency, or private registry only when the user explicitly says they are testing an unpublished fork, and label that as an internal exception.

If the npm package or CLI command is unavailable, classify it as a packaging, registry, or platform tooling regression rather than telling external creators to use a local SDK path by default.

## CLI Readiness Gate

The CLI should be able to perform the public onboarding steps it advertises:

- login
- create or update app
- configure origin and redirect URI
- write env files
- save selected app context when supported

If those commands are unavailable or fail, classify the gap as Funloom tooling readiness work. The external game integration can proceed with manually supplied values, but the creator-facing plugin should not pretend a failed CLI path succeeded.

For ordinary creators, platform developer mode is the first app-creation path. CLI app creation remains useful for automation, diagnostics, and advanced configuration, but the skill should not make CLI login/app create a required public first step when the creator already has `appId` and `appSecret` from the platform.

## Platform Authorization Gate

External game binding depends on Funloom platform authorization UX. The platform should support:

- clear display of the current Funloom account
- approve and reject
- retry after rejection
- switch account or logout during authorization
- redirect back with success, denial, or error status

If rejection followed by retry always shows the same Funloom account and there is no switch-account option, this is a platform capability gap. The external game should reset its local pending state and offer retry/cancel, but account switching belongs to Funloom.

## Platform Recharge Gate

The external unified recharge page should be platform-owned and game-agnostic:

- display the game name from the app/session, not hardcoded text
- return and display diamond balance, including `0`
- show package tiers when package tiers exist
- open QR/payment UI after package selection
- work on mobile and desktop without distorted assets
- keep platform-internal recharge separate from external-game recharge

Missing public session fields, wrong game names, or absent package tiers are OpenAPI/platform issues unless the game is using the wrong app credentials.

## Ownership Output

When finishing an integration or troubleshooting pass, separate findings into:

- game changes made
- SDK packaging or SDK API gaps
- Funloom platform capability gaps
- deployment/secret configuration required from the operator
