# Product Design Guide

Funloom integration is a product design task before it is an SDK task. The agent should help creators decide how payment, AI, and game state feel inside their game.

## Diamond Economy

Default product pattern:

```text
Funloom diamonds are the recharge/source currency.
The game consumes its own intermediate item or currency.
```

This pattern is optional, not an automatic database requirement. After scanning the project, explain whether an intermediate item is actually useful and ask the user to confirm before adding local persistence. If the game only needs Funloom balance, recharge, and Funloom official AI, prefer using Funloom platform records directly without a game-side wallet table.

Possible intermediate resources:

- Start tickets
- Energy
- Action points
- Inspiration
- NPC dialogue tokens
- Magic stones
- Draw tickets
- Premium scene unlocks

If the user confirms an intermediate resource, keep exchange ratios and prices in one config module. Do not scatter numbers through UI and server logic.

If the game already has wallet, inventory, order, or ledger storage, recommend the smallest reuse of those existing structures. New tables are for game-owned state only; Funloom-owned diamonds, payment orders, consume ledger, and AI traces remain in Funloom.

## Pricing Questions

Ask based on observed gameplay surfaces:

- If the project has a "start game" or "new run" flow, ask whether the cost belongs at game start, restart, or premium mode entry.
- If the project has AI chat/NPC replies, ask whether each AI action costs an item, has daily free uses, or is bundled into a paid session.
- If the project has inventory/wallet UI, propose placing exchange and balance there.
- If no wallet UI exists, propose a minimal exchange modal with diamond balance and recharge entry.
- Ask whether the game should persist any local paid entitlement at all. Do not assume a local item, wallet, ledger, or webhook table is needed just because Funloom payment is integrated.

## Confirmation And Failure States

Build or request these states by default:

- Before spending an item, show a confirmation dialog naming the exact cost.
- Restart/new game flows that spend again need a second confirmation.
- If item balance is insufficient, open the exchange UI.
- If diamond balance is insufficient, show a modal with "recharge" and "cancel".
- Disable repeated clicks while exchange, charge, recharge-session creation, or AI calls are pending.
- If deduction succeeds and later work fails, either roll back, compensate, or record a recoverable pending state.
- If AI fails before deduction, do not charge.

## AI Provider Switching

Preserve existing AI behavior. Add Funloom as another provider:

- `local` / existing provider
- `funloom`
- optional model or scene-level policy

Keep provider choice in configuration or settings, not hardcoded inside a prompt call.

If the game currently asks users to enter their own AI key, keep that as the self-key/local mode. The Funloom mode should represent official hosted AI: the browser chooses the mode, but the game server or serverless layer owns the Funloom SDK server client, billing, idempotency, and AI call. Do not expose official AI keys or Funloom secrets to the browser.

## Model And Capability Questions

After scanning the project, ask model questions using the game's real surfaces instead of generic AI wording:

- Channel: Funloom official AI currently has two intended channels to consider: Doubao official Volcengine and the ggb proxy. Default to recommending ggb proxy unless the creator or platform policy says otherwise.
- Text: Which provider/model or Funloom scene policy should power chat, narrative, JSON planning, and NPC replies? Current default recommendation is DeepSeek `deepseek-v4.1-flash` via ggb.
- Image: Does the game use image generation today, and should Funloom official image generation be enabled now or left for later? Current default recommendation is ggb image2.
- Fallback: If Funloom official AI is rate-limited or unavailable, should the game retry, use templates, fall back to self-key mode, or block the action?
- Cost: Is each AI action charged directly, bundled into an item/session, or free during testing?

If the creator says Funloom should own model choice, document that as a platform-side scene policy. If the SDK does not expose a channel or image-model parameter, do not describe the SDK as having a hardcoded provider; say the effective channel/model is selected by Funloom platform configuration.
