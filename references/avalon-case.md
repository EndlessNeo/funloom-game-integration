# Avalon Case Pattern

Use this as a source of generalized patterns, not as a product template to copy. Do not carry Avalon-specific names, prices, or UI placement into other games unless their scanned project shape supports the same choices.

## Observed Shape

Avalon has a web frontend and backend, existing AI behavior, and a game-start flow. The desired Funloom integration keeps existing AI settings while adding Funloom AI as a switchable provider.

## Product Decisions In This Case

- Add a minimal external-game account when no stable account exists.
- Bind Funloom user to the external game account.
- Allow rebinding by product policy.
- Show Funloom diamond balance.
- Use an intermediate item: start ticket.
- Exchange ratio: 1 start ticket costs 10 diamonds.
- Starting a game consumes 1 start ticket.
- Restarting/new game consumes another start ticket.
- Confirm before spending a ticket.
- If tickets are insufficient, open exchange.
- If diamonds are insufficient, show recharge/cancel modal.

## Generalized Pattern

- If a game has a clear "start run", "enter match", or "premium session" gate, consider charging an intermediate ticket at that gate.
- If a game has repeated AI actions, consider per-action tokens, daily free uses, or session bundles instead.
- If a game already has inventory/wallet concepts, reuse them; if not, add the smallest exchange and balance surface that supports the paid flow.
- If an issue is caused by SDK distribution or Funloom authorization behavior, record it as SDK/platform readiness work rather than baking an Avalon-only workaround into the game.

## Implementation Shape

- Frontend adds account/binding/settings panels, balance display, exchange modal, insufficient-diamond modal, and start/restart confirmations.
- Backend owns Funloom app secret, binding, balance lookup, exchange, deduction, recharge session creation, AI provider calls, and idempotency.
- Existing AI provider config remains available; Funloom is added as another provider.

## Reusable Lesson

The plugin should not ask, "Do you want AI to deduct diamonds per call?" before inspecting gameplay. It should first identify where the game currently spends player attention or resources, then offer concrete monetization shapes grounded in those flows.
