---
name: funloom-game-integration
description: Use when integrating Funloom SDK, OpenAPI, AI provider switching, user binding, diamond balance, item exchange, recharge, payment callbacks, or webhook flows into an existing web game.
---

# Funloom Game Integration

Use this skill when helping a creator connect an existing web game to Funloom. The agent's job is not to ask abstract API questions first; it must inspect the actual project, describe what it found, then ask project-aware product and implementation questions.

## Core Rule

Every clarification must be grounded in observed project facts.

Bad:

> Do you have an account system?

Good:

> I checked the backend routes and local storage usage. This project currently has no stable player account, only a browser-side guest id, so Funloom binding would be fragile. I recommend adding a minimal phone/password account before binding. Is that acceptable for this game?

If the project cannot be inspected, say exactly why and ask for the missing path, repo, or files. Do not continue with generic onboarding questions while pretending the project has been analyzed.

## Workflow

1. Identify the target game project before running project-level commands. In multi-project workspaces, inspect the relevant frontend/backend directories separately.
2. Check framework, package manager, backend/serverless presence, env files, auth/player identity, wallet/item systems, ledger/order/payment tables, AI provider code, text/image model configuration, payment/recharge code, SPA routing/deep-link behavior, and existing Funloom SDK or CLI references.
3. Report findings in "I checked / I found / this means" language, then ask one high-impact question at a time.
4. Before proposing or creating any game-side database table, state the recommended data ownership based on the scanned project: what Funloom already records, what the game already stores, and what new local state is truly needed. Ask the user to confirm this database/ledger boundary and wait for their answer.
5. Convert technical integration into game design: where AI appears, which text/image models or Funloom platform scene policies should be used, what is paid, whether diamonds buy an intermediate item, whether that intermediate item needs local persistence, and what confirmations or recovery states users need.
6. Propose a concrete integration plan and wait for confirmation before editing code, unless the user explicitly asked to implement directly.
7. Implement with minimal scoped changes, preserving existing AI configuration and adding Funloom as a switchable provider rather than replacing local settings.
8. Verify with the smallest meaningful build, test, or route check. For browser auth/recharge flows, verify the whole redirect loop, including direct access to callback/return deep links and JS/CSS MIME types after deployment. Then explain required manual deployment or secret configuration.

## Default Assumptions

- For external creators, default to Funloom production endpoints. Do not expose test-environment choices in ordinary onboarding unless the user says they are doing internal testing.
- Public creator onboarding starts from Funloom platform developer mode: the creator creates an external game there and copies the generated `appId` and one-time `appSecret`. Do not require creators to pre-fill origin, redirect URI, webhook, or AI scene settings before inspecting the game; generate those values from the project during integration.
- `FUNLOOM_API_BASE_URL` is the Funloom OpenAPI/backend base. `FUNLOOM_FRONTEND_BASE_URL` is the Funloom platform UI base that hosts authorization and recharge pages. It is not the external game's own URL. For standard Funloom environments, the platform backend may infer the matching frontend when it creates a server-side recharge `checkoutUrl`; the Browser SDK still needs the correct frontend base when it constructs authorization or checkout URLs, and OpenAPI does not construct the authorization URL. Use explicit frontend/recharge base env only as an override for non-standard, preview, or custom-domain deployments, while still passing the frontend base to Browser SDK URL builders.
- Public creator onboarding uses the official npm packages documented in `docs/funloom-sdk`: `@funloom/sdk` for code and `@funloom/cli` for the `funloom` command. Treat local path or Git SDK dependencies as internal exceptions only when the user explicitly says they are testing an unpublished fork.
- Always add a visible recharge entry and a diamond balance surface. If the game has an inventory or wallet, integrate there; otherwise show balance in the exchange/recharge UI.
- Always separate local env, frontend build env, backend/serverless env, and deployed platform secrets. A CLI-written `.env.local` does not configure Cloudflare, Vercel, Render, or other deployed environments.
- Prefer diamonds -> in-game item/currency -> gameplay cost only after the user confirms the game needs local persisted entitlement. Do not assume AI calls should directly deduct diamonds, and do not introduce a local wallet/ledger merely because Funloom is integrated.
- Funloom is always the source of truth for Funloom diamonds, recharge orders, payment status, platform consume ledger, and Funloom AI charge traces. External games should not duplicate those tables by default.
- Reuse the game's existing account, inventory, entitlement, order, or ledger tables when they exist. Add new game-side tables only for game-owned state such as local accounts, local items, chapter unlocks, usage quotas, or recovery records, and only after asking the user to confirm that boundary.
- External games need a stable local player identity for save data, inventory, and Funloom binding. If the scanned project only has guest/local state, propose the smallest account system that fits the game before binding.
- Browser code must not receive `appSecret` or payment secrets.
- Chargeable AI, diamond deduction, item exchange, and webhook verification belong on the server or serverless side.
- All diamond deductions require an idempotency key.
- For chargeable AI, a stable `idempotencyKey` makes the wallet consume operation idempotent but does not deduplicate the model/provider call. The current flow calls the provider before consuming diamonds; handle provider success followed by consume failure as a separate API-error path.
- The current external OpenAPI binding behavior is automatic within one `appId`: auth-code exchange and direct link can replace an existing active binding. Do not promise `FUNLOOM_USER_ALREADY_LINKED`, `already_bound`, or `rebind_required` from the platform API; show the current and target game accounts and confirm before any operation that may rebind.
- Webhook events use an envelope with `id`, `type`, `appId`, `createdAt`, and `data`. Verify the raw body with `SHA256(webhookSecret)` as the HMAC key over `timestamp + "." + rawBody`; make delivery handling idempotent and return non-2xx only when a retry is intended.
- Payment provider credentials belong only to the Funloom platform backend, not external games or the SDK.
- If the Funloom authorization page cannot switch accounts after a user rejects binding, classify that as a Funloom platform capability gap. The game may offer retry/cancel and explain the current Funloom login state, but it should not fake account switching.
- Funloom official AI for external games does not mean the external game can ignore model policy. After inspecting existing AI code, explain that the external-game OpenAPI/SDK AI route may use multiple platform channels, including Doubao official Volcengine and the ggb proxy. The ggb default applies only to external-game OpenAPI/SDK calls, not to Funloom platform internal generation, creator tools, operations, or other product AI capabilities. Current external OpenAPI defaults are ggb proxy, DeepSeek text model `deepseek-v4.1-flash`, and ggb image2 for image generation. Ask the creator which text model, image model, and scene-level policy should apply unless the project or Funloom platform app already has an explicit configured policy. If the SDK/OpenAPI only exposes `model` or `sceneCode` and not a channel selector, state that provider/channel routing is platform-side configuration, not a browser/game-side secret.
- Do not expose raw upstream model errors to players. Classify 429/rate-limit/provider errors as model/platform capacity or configuration issues, map them to friendly UI text, disable repeated submits while pending, and verify failed model calls do not charge diamonds.

## Read References As Needed

- For the discovery checklist and project-aware question patterns, read [references/project-aware-discovery.md](references/project-aware-discovery.md).
- For SDK, OpenAPI, environment, recharge, webhook, and payment separation rules, read [references/integration-rules.md](references/integration-rules.md).
- For SDK packaging readiness and Funloom platform capability gaps, read [references/sdk-and-platform-readiness.md](references/sdk-and-platform-readiness.md).
- For local/deployed env, current-environment diagnosis, CLI env files, and Cloudflare-style deployment warnings, read [references/deployment-and-env.md](references/deployment-and-env.md).
- For account systems, external user ids, rebinding, duplicate registration, and cancellation states, read [references/identity-and-binding.md](references/identity-and-binding.md).
- For external recharge page behavior, package tiers, QR flow, balance display, and visual QA, read [references/external-recharge-ux.md](references/external-recharge-ux.md).
- For product design prompts around diamonds, intermediate items, AI pricing, confirmation dialogs, and failure handling, read [references/product-design.md](references/product-design.md).
- For common failures such as wrong app name, balance `--`, recharge 500, binding conflicts, and environment confusion, read [references/troubleshooting.md](references/troubleshooting.md).
- For the Avalon reference implementation pattern, read [references/avalon-case.md](references/avalon-case.md).

## Red Flags

Stop and re-scan before continuing if you catch yourself doing any of these:

- Asking "do you have X?" before checking whether X exists in the project.
- Deleting or overwriting the game's existing AI configuration instead of adding a switch.
- Putting Funloom `appSecret`, webhook secrets, or payment keys in browser code.
- Falling back to local path or Git SDK dependencies for public creator onboarding when the official npm packages are available.
- Treating CLI app creation as the public first step when the creator can create the app in Funloom platform developer mode.
- Assuming `.env.local`, `.env.example`, or CLI output has configured the deployed frontend/backend.
- Hiding which Funloom environment a game is connected to when diagnosing login, recharge, or callback bugs.
- Allowing login, bind, authorize, recharge, exchange, or payment buttons to accidentally submit forms or refresh the game page.
- Treating Funloom diamonds as the only in-game economy without discussing game-specific items or currency.
- Creating game-side wallet, order, payment, ledger, idempotency, or webhook tables before asking the user to confirm what should live in Funloom versus the game.
- Duplicating Funloom platform diamonds, payment orders, consume ledger, or AI trace data in the external game when the game has no local entitlement to persist.
- Letting zero balances render as unknown values such as `--`.
- Letting a recharge button submit a form or refresh the page instead of creating/opening a recharge session intentionally.
- Fixing recharge "refreshes the game page" only by hardcoding `FUNLOOM_FRONTEND_BASE_URL` in `toml` or deployed secrets when the standard OpenAPI host should automatically produce the matching absolute checkout URL.
- Routing an external game recharge through the platform's internal recharge endpoint.
- Treating missing Funloom switch-account/logout behavior as an external game bug.
- Asking users to delete database rows manually instead of designing account recovery or a confirmed rebind flow.
- Silently triggering the current automatic Funloom rebind without showing the current and target game accounts or preserving an old-account recovery path.
- Telling creators to choose between production and test endpoints during normal public onboarding.
