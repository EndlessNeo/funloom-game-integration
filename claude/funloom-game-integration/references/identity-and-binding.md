# Identity And Binding

Use this when the target game lacks stable accounts, has duplicate registration behavior, or users hit Funloom binding conflicts.

## Two Account Layers

Explain the distinction clearly:

- Funloom account: platform identity, authorization, diamonds, recharge.
- External game account: save data, inventory, entitlement, local economy, and the binding anchor.

Funloom accounts do not replace the game's need for a stable player identity when the game stores progress, items, tickets, or paid state.

## Minimal Account Fallback

If the scanned game only has local guest state and paid state must persist, propose the smallest account system that fits the product:

- phone or email login if the project needs recovery
- password or one-time-code depending on existing auth capability
- unique user record
- session/JWT secret configured per deployment
- account settings surface for binding status

Do not force a large auth system if a smaller stable identity is enough, but do not bind paid state to a volatile local storage id.

## externalUserId Rules

The binding key should be scoped by Funloom app:

```text
unique(appId, externalUserId)
```

Use a stable game-side user id. Avoid naked global autoincrement ids in public guidance unless the backend scopes them by app id. Do not use phone numbers, emails, nicknames, or browser-only guest ids as the OpenAPI identity unless the project explicitly accepts those privacy and stability tradeoffs.

## Current Funloom Binding Behavior

In the current external OpenAPI implementation, auth-code exchange and direct binding are automatic within one `appId`:

- an existing active link for the same Funloom user can be replaced by the new external user
- an existing active link for the same external user can be updated to the new Funloom user
- the API does not currently return `FUNLOOM_USER_ALREADY_LINKED`, `already_bound`, or `rebind_required`

Because this can move the platform binding, the game should show the current and target game accounts and require confirmation before calling a flow that may rebind. Preserve an old-account recovery path and never ask ordinary users to delete database rows manually.

## Idempotent Account And Binding States

The game's own account registration and login may return explicit states such as:

- `already_registered` when the same credential maps to an existing game account
- `cancelled` or `denied` when the user refuses authorization

Do not let duplicate registration create a new game user that silently loses the previous Funloom binding or paid inventory. Treat the current Funloom automatic rebind as a state-changing operation, not as an API conflict that the game can wait to receive.

## Cancel, Reject, Retry

Handle authorization outcomes explicitly:

- approved: persist binding and refresh balance
- rejected/cancelled: clear local pending state and return to unbound UI
- error: show retry and diagnostics
- binding may replace an active link: show the target account and require confirmation before retrying

If switching Funloom accounts is required, that capability belongs on the Funloom authorization page. The external game should not clear Funloom cookies or pretend it can change platform login state.
