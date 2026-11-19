---
name: funloom-game-integration
description: Use when integrating a web or H5 game with Funloom, especially when choosing between Hosted SDK 2.0 for Funloom-hosted games and Browser/Server SDK with OpenAPI for externally deployed games.
---

# Funloom Game Integration

Use this skill when helping a creator connect a web or H5 game to Funloom. Select the integration by the intended runtime, not by whether the project is H5, static, or already has a backend.

## Choose The Runtime First

Before recommending an SDK or designing integration work, establish where the game will run:

- If the game package will be uploaded to Funloom Advanced Custom Game mode and played in Funloom's hosted iframe, use Hosted SDK 2.0 (`@funloom/sdk/hosted`). This applies to static H5 games and games that also have their own backend.
- If the game will run on the developer's own site, server, or another platform, use the external-game Browser/Server SDK and OpenAPI flow described in this skill's existing integration references.
- If the runtime is unclear, inspect the project and explain the relevant options, then ask one direct question: "Will this game package run in Funloom Advanced Custom Game mode, or on your own site/server?" A static H5 project is a reason to recommend hosted mode, not to assume it.

For Hosted games, do not introduce external-game OAuth, `appSecret`, Browser/Server SDK clients, external-user binding, or OpenAPI callbacks for capabilities already provided by the host. The game may retain its own backend for game-owned business data, but Funloom player identity, cloud saves, configured items, AI, wallet, and recharge must use the Hosted bridge when enabled. Direct local HTML or an ordinary local dev server is not a valid Hosted session; end-to-end capability testing must enter through Funloom's creator test flow.

Read [references/hosted-sdk-2.md](references/hosted-sdk-2.md) for Hosted-specific implementation and testing guidance, including the required configuration-driven item identity, display, exchange ratio, and the separation between platform exchange quantity and game-defined gameplay cost. Do not apply the external-game discovery checklist or external integration references to a confirmed Hosted-only request unless a distinct external deployment is also in scope.

## Hosted Configuration Contract

For Hosted games, every platform-controlled item field must come from the current `configuration.get()` result:

- Treat `itemId`, `name`, `price`, and `quantity` as one configuration snapshot. Use that same snapshot for player-facing copy, inventory lookup, exchange calculations, request parameters, success/error messages, and transaction metadata.
- Do not keep a hardcoded item ID/name/price/quantity as a fallback after Hosted initialization succeeds. If configuration is missing, invalid, or ambiguous, block the item action and show a recoverable configuration error.
- Refresh configuration before a paid or consuming workflow when stale configuration could charge the wrong offer. The public `@funloom/sdk@0.1.2` `HostedItemOptions` only exposes `quantity`, `requestId`, and `signal`; do not add `expectedPrice` or `expectedQuantity` to ordinary game code. If a specific platform-host extension supports those fields, label them as host-side contract fields and handle `PRICE_CHANGED` by discarding the stale request payload, loading the latest configuration, and retrying with a new request ID.
- Hardcode only game-owned rules that the platform does not configure, such as `GAMEPLAY_START_COST = 5`. Never reuse the platform bundle `quantity` as the gameplay cost.
- Treat Hosted `chargedUnits` as a protocol value: for KK, it is minor units where `100` means `1 KK币`; convert it before player-facing display. Creator-test points are whole points and must not use the KK conversion.
- Treat `EXCHANGE_CANCELED` as a user decision, not a failed exchange. The host owns the confirmation UI and may provide one lightweight cancellation toast; game code should close/reset its pending state without adding generic failure, retry, or recharge copy for this code.
- Persist transaction-time snapshots for financial activity: game ID/title, item ID/name, total item units, units per bundle, bundle count, price per bundle, actual charged currency/amount, request ID, and mode. Historical activity copy must use the snapshot first so later configuration edits do not rewrite history.
- Activity copy must state what was exchanged and charged. Do not use generic text such as "Hosted 游戏道具兑换" or "xx变动" when transaction details are available.

For Hosted AI:

- Enable AI once for the game, then configure as many stable `sceneCode` entries as the game needs. Each entry maps a game-owned call label to one platform-approved model; multiple entries may share a model.
- The game owns its system prompt, conversation context, and call timing. Pass `system`, `user`, and `assistant` messages through `ai.chat()`; do not expect the platform settings page to store or return a system prompt.
- Funloom owns model availability, output limits, provider routing, usage measurement, and the current player's actual-usage charge. Hosted game code must not select a provider, model, or charge amount at runtime.
- `sceneCode` is a routing and audit label, not a platform-authored gameplay scenario. Keep it stable and use a new `requestId` for a new mutation or retry the same request ID only with the identical payload.

## Core Rule

Every clarification must be grounded in observed project facts.

Bad:

> Do you have an account system?

Good:

> I checked the backend routes and local storage usage. This project currently has no stable player account, only a browser-side guest id, so Funloom binding would be fragile. I recommend adding a minimal phone/password account before binding. Is that acceptable for this game?

If the project cannot be inspected, say exactly why and ask for the missing path, repo, or files. Do not continue with generic onboarding questions while pretending the project has been analyzed.

## Workflow

1. Identify the target game project before running project-level commands. In multi-project workspaces, inspect the relevant frontend/backend directories separately.
2. Establish the intended runtime using "Choose The Runtime First". If it is Hosted, follow the Hosted-specific reference and scan only what is relevant to the game package and any game-owned backend. If externally deployed, continue with the external-game checklist below.
3. For external games, check framework, package manager, backend/serverless presence, env files, auth/player identity, wallet/item systems, ledger/order/payment tables, AI provider code, text/image model configuration, payment/recharge code, SPA routing/deep-link behavior, and existing Funloom SDK or CLI references.
4. Report findings in "I checked / I found / this means" language, then ask one high-impact question at a time.
5. Before proposing or creating any game-side database table for an external game, state the recommended data ownership based on the scanned project: what Funloom already records, what the game already stores, and what new local state is truly needed. Ask the user to confirm this database/ledger boundary and wait for their answer.
6. For external games, convert technical integration into game design: where AI appears, which text/image models or Funloom platform scene policies should be used, what is paid, whether diamonds buy an intermediate item, whether that item needs local persistence, and what confirmations or recovery states users need.
7. Propose a concrete integration plan and wait for confirmation before editing code, unless the user explicitly asked to implement directly.
8. Implement with minimal scoped changes, preserving existing AI configuration and adding Funloom as a switchable provider rather than replacing local settings.
9. Verify with the smallest meaningful build, test, or route check. For external browser auth/recharge flows, verify the whole redirect loop, including direct access to callback/return deep links and JS/CSS MIME types after deployment.
10. After a Hosted integration builds successfully, prepare the upload deliverable before reporting completion. Create a version-labeled ZIP when the platform accepts ZIP uploads; otherwise create a version-labeled clean upload directory and tell the creator exactly which files to upload.
11. For every configuration-driven Hosted item, verify at least two platform configurations that change the bundle quantity, such as 5 and 10. Confirm the visible exchange ratio, calculated bundle count, exchange result, inventory, consume request, and activity detail all use the selected configuration while game-owned costs stay unchanged.
12. Report the upload artifact path, included file list, excluded development files, size checks, and any manual platform configuration or test steps.

## Hosted Upload Deliverable

The Hosted game upload is a runtime package, not the development project directory. Always produce a clean, versioned artifact after implementation.

- Include the root `index.html`, the JavaScript and CSS files it references, the bundled Hosted SDK runtime, every referenced runtime asset under `assets/` or equivalent, and every local module imported by those runtime JavaScript files.
- Exclude `node_modules/`, `.git/`, source files that are not referenced at runtime, test and scratch directories, `package.json`, lockfiles, local SDK `.tgz` files, build tools, source maps unless the platform explicitly requires them, `.env*`, secrets, and documentation files.
- Put `index.html` at the ZIP root. Do not create an extra top-level project directory inside the archive.
- Use a deterministic version label such as `<game-name>-funloom-hosted-vYYYYMMDD-HHmm.zip`. Use the local project timezone for the label and avoid spaces in the filename.
- Check every included file's uncompressed size before packaging. If the platform has a per-file limit, no included file may exceed that limit; ZIP compression does not make an oversized individual file acceptable.
- After creating the ZIP, list its entries and inspect the archive in a fresh temporary directory. Confirm that `index.html` is at the root, all HTML asset references and transitive local JavaScript imports resolve to files in the archive, no excluded development files are present, and no file exceeds the platform limit.
- Search the packaged runtime for obsolete platform item IDs, names, prices, quantities, and copy. A successful source build is not enough if the uploaded directory or ZIP still contains an older runtime.
- Remove temporary extraction and verification directories before final reporting. Keep the final ZIP in the project's designated output directory or another ignored temporary/output location, and do not add it to source control unless the creator explicitly requests that.

For a static game without a build directory, use an explicit allowlist based on the HTML references rather than uploading the whole project. A typical package contains:

```text
index.html
index.js
index.css
funloom-sdk.js
assets/
```

Do not upload the local SDK tarball or `node_modules`; the SDK must already be bundled into the browser file used by `index.html`.

## Default Assumptions

The following defaults apply to externally deployed games. A confirmed Hosted game follows the Hosted rules above and does not need external app credentials, OAuth, Browser/Server clients, or OpenAPI callbacks for host-provided capabilities.

- For external creators, default to Funloom production endpoints. Do not expose test-environment choices in ordinary onboarding unless the user says they are doing internal testing.
- Public creator onboarding starts from Funloom platform developer mode: the creator creates an external game there and copies the generated `appId` and one-time `appSecret`. Do not require creators to pre-fill origin, redirect URI, webhook, or AI scene settings before inspecting the game; generate those values from the project during integration.
- `FUNLOOM_API_BASE_URL` is the Funloom OpenAPI/backend base. `FUNLOOM_FRONTEND_BASE_URL` is the Funloom platform UI base that hosts authorization and recharge pages. It is not the external game's own URL. For standard Funloom environments, the platform backend may infer the matching frontend when it creates a server-side recharge `checkoutUrl`; the Browser SDK still needs the correct frontend base when it constructs authorization or checkout URLs, and OpenAPI does not construct the authorization URL. Use explicit frontend/recharge base env only as an override for non-standard, preview, or custom-domain deployments, while still passing the frontend base to Browser SDK URL builders.
- Public creator onboarding uses the official npm packages documented in `docs/funloom-sdk`: `@funloom/sdk` for code and `@funloom/cli` for the `funloom` command. Treat local path or Git SDK dependencies as internal exceptions only when the user explicitly says they are testing an unpublished fork.
- For external games, always add a visible recharge entry and a KK Coin balance surface. If the game has an inventory or wallet, integrate there; otherwise show balance in the exchange/recharge UI. Hosted games should use the host-provided `wallet.balance()` and `recharge.open()` capabilities when enabled.
- Always separate local env, frontend build env, backend/serverless env, and deployed platform secrets. A CLI-written `.env.local` does not configure Cloudflare, Vercel, Render, or other deployed environments.
- Prefer diamonds -> in-game item/currency -> gameplay cost only after the user confirms the game needs local persisted entitlement. Do not assume AI calls should directly deduct diamonds, and do not introduce a local wallet/ledger merely because Funloom is integrated.
- Funloom is always the source of truth for Funloom diamonds, recharge orders, payment status, platform consume ledger, and Funloom AI charge traces. External games should not duplicate those tables by default.
- Reuse the game's existing account, inventory, entitlement, order, or ledger tables when they exist. Add new game-side tables only for game-owned state such as local accounts, local items, chapter unlocks, usage quotas, or recovery records, and only after asking the user to confirm that boundary.
- External games need a stable local player identity for save data, inventory, and Funloom binding. If the scanned project only has guest/local state, propose the smallest account system that fits the game before binding.
- Browser code must not receive `appSecret` or payment secrets.
- For external games, chargeable AI, KK Coin deduction, item exchange, and webhook verification belong on the server or serverless side. Hosted games use the host bridge and its `requestId` contract instead.
- All external-game KK Coin deductions require an idempotency key; Hosted mutations use stable `requestId` values.
- For chargeable AI, a stable `idempotencyKey` makes the wallet consume operation idempotent but does not deduplicate the model/provider call. The current flow calls the provider before consuming diamonds; handle provider success followed by consume failure as a separate API-error path.
- The current external OpenAPI binding behavior is automatic within one `appId`: auth-code exchange and direct link can replace an existing active binding. Do not promise `FUNLOOM_USER_ALREADY_LINKED`, `already_bound`, or `rebind_required` from the platform API; show the current and target game accounts and confirm before any operation that may rebind.
- Webhook events use an envelope with `id`, `type`, `appId`, `createdAt`, and `data`. Verify the raw body with `SHA256(webhookSecret)` as the HMAC key over `timestamp + "." + rawBody`; make delivery handling idempotent and return non-2xx only when a retry is intended.
- Payment provider credentials belong only to the Funloom platform backend, not external games or the SDK.
- If the Funloom authorization page cannot switch accounts after a user rejects binding, classify that as a Funloom platform capability gap. The game may offer retry/cancel and explain the current Funloom login state, but it should not fake account switching.
- Funloom official AI for external games does not mean the external game can ignore model policy. After inspecting existing AI code, explain that the external-game OpenAPI/SDK AI route may use multiple platform channels, including Doubao official Volcengine and the ggb proxy. The ggb default applies only to external-game OpenAPI/SDK calls, not to Funloom platform internal generation, creator tools, operations, or other product AI capabilities. Current external OpenAPI defaults are ggb proxy, DeepSeek text model `deepseek-v4.1-flash`, and ggb image2 for image generation. Ask the creator which text model, image model, and scene-level policy should apply unless the project or Funloom platform app already has an explicit configured policy. If the SDK/OpenAPI only exposes `model` or `sceneCode` and not a channel selector, state that provider/channel routing is platform-side configuration, not a browser/game-side secret.
- Do not expose raw upstream model errors to players. Classify 429/rate-limit/provider errors as model/platform capacity or configuration issues, map them to friendly UI text, disable repeated submits while pending, and verify failed model calls do not charge diamonds.

## Read References As Needed

- For Funloom-hosted Advanced Custom Games using SDK 2.0, read [references/hosted-sdk-2.md](references/hosted-sdk-2.md).
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

- Choosing an SDK based only on "H5", static files, or presence of a custom backend instead of where the game will run.
- Applying OAuth, `appSecret`, external-user binding, or OpenAPI server flows to a game confirmed to run in Funloom Advanced Custom Game mode.
- Treating an existing game backend as a reason to avoid Hosted SDK 2.0 for Funloom-hosted player/session capabilities.
- Telling a Hosted game to test its real host capabilities by opening the HTML directly or using an ordinary local dev server.
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
