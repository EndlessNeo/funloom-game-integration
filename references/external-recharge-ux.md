# External Recharge UX

Use this when integrating or reviewing the Funloom unified recharge page from an external game.

## Expected Flow

The external game should:

1. Create a recharge session from its backend/serverless layer.
2. Open the Funloom external unified recharge page for that session.
3. Let the platform page show packages and create payment QR/payment UI.
4. Refresh game balance after return, polling, or webhook reconciliation.

The game should not call platform-internal payment endpoints directly and should not hold payment provider credentials.

The OpenAPI session response should provide an absolute `checkoutUrl`. In standard Funloom test and production environments, that URL should be derived from the active OpenAPI host on the platform side, not from a creator-maintained game config switch. If a relative path such as `/funloom/external/recharge?...` reaches the browser, treat it as a platform/OpenAPI integration defect to fix at the source; the game may still use the SDK's checkout opener or URL builder as a defensive fallback for older responses.

## Session Data Contract

The public recharge session should provide enough non-secret data for a game-agnostic page:

- game/app name
- diamond balance, including `0`
- package tiers or product list
- selected currency/amount labels
- session status
- return/cancel URLs when supported

If package tiers or balance are missing from the public session, classify it as an OpenAPI/platform issue unless the game created the session with wrong app credentials.

## Button And Navigation Safety

Recharge, login, bind, authorize, exchange, and payment buttons inside forms must not accidentally submit or refresh the game page. Use `type="button"` or `preventDefault` where appropriate. Disable repeated clicks while creating sessions or submitting mutations, and surface failure with retry/cancel.

## Visual And Responsive QA

The platform-owned page should be polished enough for many games:

- show package tiers by default when available
- open QR/payment UI after package selection
- work on mobile and desktop
- avoid asset distortion with fixed aspect ratios and `object-fit`
- keep titles and balances readable without dominating the screen
- show `0` as `0`, not `--`
- show diagnostic empty states when package tiers are missing
- hide or style scrollbars without breaking scroll
- avoid hardcoded game names or game-specific copy

External games can theme entry points, but the unified checkout itself should stay game-agnostic.

## Balance Refresh

Do not rely only on the page opening successfully. After payment success:

- process webhook idempotently
- refresh or poll balance/session state
- update game inventory or intermediate currency
- handle pending/cancelled/failed states clearly
