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

## Conflict And Rebind

When a binding conflict occurs:

- identify whether the Funloom user is bound in the same app or another app
- identify whether the previous game account still exists
- offer account recovery if the old game account exists
- offer explicit rebind if product policy allows it
- show the current and target account labels before rebinding
- never ask ordinary users to delete database rows manually

## Idempotent Account And Binding States

Account registration, login, binding, and rebinding should return explicit states the game can handle:

- `already_registered` when the same credential maps to an existing game account
- `already_bound` when the Funloom user is already bound under the relevant app scope
- `cancelled` or `denied` when the user refuses authorization
- `rebind_required` when policy allows moving the binding but needs confirmation
- `binding_conflict` when automatic recovery is unsafe

Do not let duplicate registration create a new game user that silently loses the previous Funloom binding or paid inventory.

## Cancel, Reject, Retry

Handle authorization outcomes explicitly:

- approved: persist binding and refresh balance
- rejected/cancelled: clear local pending state and return to unbound UI
- error: show retry and diagnostics
- already bound: route to recovery or rebind

If switching Funloom accounts is required, that capability belongs on the Funloom authorization page. The external game should not clear Funloom cookies or pretend it can change platform login state.
