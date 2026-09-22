# Integration Rules

Use official Funloom SDK/OpenAPI documentation supplied by the workspace or user when available. Do not invent install commands, package names, or endpoint shapes.

## SDK Distribution

Current public creator guidance starts with Funloom platform developer mode to create the external game and copy the generated `appId` plus one-time `appSecret`. After the target project has been inspected, use the official npm packages documented in `docs/funloom-sdk`:

- `npm install @funloom/sdk`
- `npm install -g @funloom/cli`

The current public docs list `@funloom/sdk@0.1.1` and `@funloom/cli@0.1.1`. If the workspace provides newer Funloom SDK docs, treat those docs as the source of truth for package versions and method names.

Use the target project's package manager and update its lockfile. Do not present internal local paths, absolute workspace paths, or Git dependencies as the normal creator experience unless the user explicitly says they are testing an unpublished SDK fork. If npm installation or CLI commands fail, classify that as a packaging, registry, or platform tooling regression to diagnose, not as evidence that public onboarding should fall back to local paths.

The same rule applies to the Funloom CLI. The published package is `@funloom/cli`, and the command is `funloom`. Treat it as a Codex automation, diagnostics, or advanced configuration tool unless the creator explicitly wants a CLI-first flow.

## Environment Policy

For public creator guidance, default to production. Do not introduce test endpoints unless the user explicitly says this is internal testing.

Separate local and deployed configuration:

- Local `.env.local` is for local dev.
- Deployed frontend/serverless secrets must be set on the hosting provider.
- Changing from test to production should be a config change, not a code rewrite.
- Frontend build-time variables and backend runtime variables are different surfaces.
- `.env.example` documents required values but is not loaded at runtime unless the project explicitly copies it.
- CLI `--write-env` can write local files, but it does not update deployed secrets.

Generated or app-specific values:

- Funloom app id
- Funloom app secret
- allowed origin
- redirect URI
- webhook secret, if issued by Funloom

The app id and app secret normally come from Funloom platform developer mode. Allowed origin, redirect URI, recharge return URL, webhook URL, and AI scene codes should be generated from the scanned game project and confirmed with the creator before being written into the project or platform configuration.

Fixed or platform values:

- production Funloom API base
- production Funloom frontend base
- standard callback/webhook route shapes

Manual host/platform secrets:

- external game's own JWT/session secrets
- Cloudflare/Vercel/Render environment variables
- Funloom platform payment secrets, only in Funloom backend deployments

When a creator asks "which Funloom environment am I connected to?", inspect and report the configured API base, frontend base, app id suffix, origin, redirect URI, and deployment target. Do not guess from the browser URL alone.

## Browser vs Server

Browser:

- use `createFunloomBrowserClient`
- may hold public app id and frontend/API base if intended public
- must not hold `appSecret`
- must not call privileged deduction or webhook verification directly

Server/serverless:

- use `createFunloomServerClient`
- stores app secret
- creates recharge sessions
- performs diamond deduction and item exchange
- calls chargeable AI
- verifies webhook signatures

## Data Ownership And Database Minimization

Funloom must keep its own platform records for diamonds, recharge orders, payment status, wallet consumption, idempotency, and AI charge traces. External games should treat those Funloom records as the source of truth for Funloom-owned money and diamond state.

Do not create game-side wallet, payment, order, ledger, webhook, or idempotency tables by default. First inspect the actual project and identify:

- whether the game already has user, inventory, entitlement, order, or ledger tables
- whether the requested feature creates game-owned state, such as local items, chapter unlocks, usage quotas, tickets, inspiration, or recovery records
- whether the flow can rely on Funloom SDK queries and platform records without local persistence

Before adding any new game-side database structure, recommend the smallest data boundary and ask the user to confirm it. The recommendation should be project-aware:

- If the game already has a ledger or inventory table, prefer extending or reusing it.
- If the game has no local paid entitlement, do not add a local wallet/ledger just to mirror Funloom.
- If the game introduces an intermediate item or persistent entitlement, store only that game-owned state locally and keep Funloom diamonds/payment records in Funloom.
- Webhooks are required only when the game needs local fulfillment, reconciliation, or recovery; pure recharge/balance display can start with SDK/server queries.

## AI Provider Switching

If the game already lets users enter their own AI key, preserve that local/self-key mode. Add Funloom as a separate official/provider mode rather than replacing the existing configuration.

- Existing user-provided keys may stay in the game's current local config if that is already the product behavior.
- Funloom official AI must go through the game server or serverless layer with `@funloom/sdk/server`; never put an official AI key, `appSecret`, or chargeable AI call in browser code.
- Server calls should use `funloom.ai.chatCompletions()` with a scene-specific `sceneCode`, explicit `diamondCost` or product-controlled cost, and a stable `idempotencyKey`.
- The browser may expose a provider switch such as `local` / `funloom`, but it should call the game backend for Funloom mode.
- Before wiring Funloom official AI, identify every existing AI surface: text chat, narrative generation, structured JSON generation, image generation, voice, embeddings, moderation, or reranking. Ask the creator which surfaces should move to Funloom and which should stay self-key/local.
- Explain the available Funloom official AI channels before asking: Doubao official Volcengine and the ggb proxy. Current default recommendation is ggb proxy for official AI, DeepSeek text model `deepseek-v4.1-flash`, and ggb image2 for image generation.
- Ask for the desired channel, text model, image model, and fallback behavior unless an explicit Funloom platform scene policy already exists for the app. Do not assume the SDK chooses Doubao, DeepSeek, OpenAI, ggb, or any other provider by itself.
- If OpenAPI only accepts `sceneCode` and/or `model` and not a provider/channel id, explain that provider/channel selection is controlled by Funloom platform configuration for that scene or model mapping. The game should still name scene codes clearly and document which gameplay action each scene charges.
- If the game needs Funloom official image generation, verify the SDK/OpenAPI actually exposes an image endpoint. If it does not, classify image generation as a Funloom platform/API capability gap or route it through an approved game backend endpoint; do not invent SDK methods.
- Add user-facing handling for rate limits and provider errors. A 429 from an upstream model provider is not an SDK URL or binding bug by default; prevent repeated submits and show a short retry message instead of raw provider JSON.

## Recharge Separation

Do not mix platform-internal recharge with external-game recharge.

- Platform internal recharge can use routes such as `/api/pay/create-native`.
- External game unified recharge should use the OpenAPI/public recharge session flow, for example a session payment route under `/openapi/v1/public/recharge/sessions/:sessionId/payment`.

External games and SDKs should not receive WeChat or payment provider credentials. If QR creation fails with server 500, first inspect the Funloom platform backend deployment secrets and notify URL.

## Webhook Rules

- Verify signature before trusting payloads.
- Make handlers idempotent.
- Store event ids or order ids when needed to prevent duplicate effects.
- Refresh or reconcile player state after payment success rather than relying only on frontend redirects.
