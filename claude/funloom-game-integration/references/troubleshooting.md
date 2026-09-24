# Troubleshooting

Use this when the integration partly works but a Funloom login, binding, balance, recharge, payment, or AI flow behaves incorrectly.

## Wrong Environment

Symptoms:

- Local game opens the wrong Funloom page.
- Deployed game still points to a test domain.
- The same app behaves differently locally and online.

Check local env, frontend build env, backend/serverless env, and deployment dashboard secrets separately. Local files do not automatically change deployed Workers/Pages/Vercel/Render variables.

For deeper diagnosis, use [deployment-and-env.md](deployment-and-env.md) and produce a safe environment fingerprint instead of guessing.

## Wrong Game Name On Recharge Page

If a recharge page shows the wrong game name, inspect the recharge session response and the app credentials used to create it. The displayed name should come from the Funloom external app associated with the app id/secret. If every game displays the same name, the integration is probably reusing one app credential pair.

## Balance Shows Unknown

If balance renders as `--`, check whether the session or balance endpoint returns a balance field. If the backend returns `0`, the frontend must display `0`, not fallback to `--`.

Rule:

```ts
balance == null ? "--" : String(balance)
```

Do not use:

```ts
balance || "--"
```

If the API truly does not return balance, classify it as a public session/OpenAPI gap. If it returns `0` but the UI shows `--`, classify it as a frontend integration bug.

## Recharge Button Refreshes The Game

Inspect whether the button is inside a form or anchor flow. Use `type="button"` or prevent default submit behavior. The intended flow is create session -> open external recharge page, not page refresh.

## Recharge 500

For external recharge QR creation failures, first inspect the Funloom platform backend logs and payment secrets.

Likely missing or wrong:

- `WX_APPID`
- `WX_API_V3_KEY`
- `WX_MCHID`
- `WX_SERIAL_NO`
- `WX_CERT_PEM`
- `WX_KEY_PEM`
- notify URL for the current environment

Do not ask the external game to provide payment keys.

If the same payment flow works in production but not in staging/test, compare platform backend secrets and notify URLs before editing SDK or external game code.

## Automatic Rebind / Binding Moved

The current OpenAPI binding endpoints do not return an "already bound" conflict. They may automatically replace an active link within the same `appId`. If a Funloom user appears to move between game accounts:

- Check whether binding is scoped by app id.
- Check whether the previous external user still exists.
- Show the target account and consequences before rebinding.
- Preserve an old-account recovery path and record the new mapping after the operation.
- Do not ask users to edit or delete database rows manually.

## Rejected Binding Opens The Same Funloom Account

If the user rejects binding, clicks bind again, and still sees the same Funloom account, do not treat this as a game-specific bug by default. The active Funloom login state is controlled by the Funloom platform authorization page.

Expected platform capabilities:

- Show the currently logged-in Funloom account clearly.
- Provide switch-account or logout during authorization.
- Return a clear denied/cancelled result to the external game.
- Let the game retry binding without stale pending state.

Game-side handling:

- Reset local "binding in progress" state after denial.
- Offer retry and cancel.
- If the platform has no switch-account/logout route, explain that account switching must happen on Funloom and list it as a platform capability gap.
- Do not clear or spoof Funloom platform cookies from the external game.

## CLI Authorization Blank Page

Check the CLI authorize URL, Funloom frontend route, login state, redirect URI, and local callback server. If the page is under the game dev server rather than the Funloom frontend, inspect the configured Funloom frontend base URL.

## OAuth Return Blank Page

If authorization succeeds but the external game is blank after returning to a callback URL, inspect the browser console and network panel before blaming the SDK or Funloom platform.

Strong signal for a game-side SPA build/routing bug:

- Console says a module script expected JavaScript but received `text/html`.
- The callback document returns 200, but a JS or CSS asset URL also returns the app's `index.html`.
- The failing asset path is nested under the callback path, such as `/funloom/assets/...`, while built assets actually live at `/assets/...`.

Common fix: adjust the frontend build base or Worker/Pages asset routing so callback and recharge-return deep links load absolute JS/CSS asset URLs. Then verify the deployed callback URL directly and confirm JS/CSS MIME types.

Do not classify this as a Funloom authorization problem when the code was issued and the failure happens while loading the external game's assets.

## AI Provider Rate Limit Or Upstream Error

If the game receives errors such as `RateLimitExceeded`, `EndpointRPMExceeded`, `TooManyRequests`, or `429` from Doubao/OpenAI/another model provider, classify the primary issue as model provider/platform capacity or scene configuration unless tracing proves the request was malformed by the game or SDK.

Check:

- Which Funloom API base and app id were used.
- Which Funloom AI channel/provider was selected, such as Doubao official Volcengine or the ggb proxy.
- Which model was selected. If none was passed by the game, inspect the Funloom platform default or backend fallback rather than calling it an SDK default. For the ggb proxy, confirm the configured text model is available there; the current external-game default is `deepseek-v4.1-flash`.
- Which game action and `sceneCode` triggered the call.
- Whether the Funloom platform AI trace recorded a provider error.
- Whether diamonds were charged despite the provider failure.
- Whether the game allowed repeated clicks or concurrent calls.

Game-side fixes should be scoped to UX and resilience: disable duplicate submits, show a friendly retry message, optionally provide template/self-key fallback if product-approved, and avoid displaying raw provider JSON to players. Platform-side fixes include increasing provider quotas, changing scene model/provider, or normalizing provider 429 responses to a clear Funloom error code such as `rate_limited`.

## Duplicate Registration Then Binding Conflict

If a user registers what appears to be the same external account again and then binding fails, inspect the external game account uniqueness rules. The durable fix is account uniqueness, recovery, or rebind. Manual database deletion is a last-resort operator action, not a creator-facing flow.

## Issue Ownership

When diagnosing an integration bug, classify it before patching:

- SDK issue: helper builds the wrong URL, exposes secrets, lacks a needed method, or mismatches OpenAPI.
- OpenAPI/platform issue: required public session fields are missing, authorization lacks switch account, recharge page lacks package selection, payment secrets are missing, or webhook signing is incomplete.
- Game integration issue: wrong app credentials, wrong environment variables, wrong endpoint, missing backend proxy, UI fallback bugs, or stale local state.

Report this classification to the creator so platform fixes do not get hidden as one-off game patches.
