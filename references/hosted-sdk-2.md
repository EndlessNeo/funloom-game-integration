# Hosted SDK 2.0: Funloom Advanced Custom Games

Use this path only when the game package is uploaded to Funloom Advanced Custom Game mode and runs inside the Funloom-hosted iframe. The deciding factor is the runtime location, not the game framework, static-vs-dynamic architecture, or presence of a separate game backend.

## Integration Boundary

- Install and bundle `@funloom/sdk`, importing the Hosted entry from `@funloom/sdk/hosted`.
- Connect with `connectHostedGame()` and use the host session for Funloom player identity and enabled platform capabilities.
- Do not create an external-game OAuth flow, use `appSecret`, initialize Browser/Server SDK clients, or implement external-user binding for the Hosted session.
- Keep a custom backend only for game-owned services or data that the game itself must own. Do not proxy Hosted capabilities through it or duplicate Funloom's wallet, recharge orders, AI charges, or host-managed saves.
- Hosted SDK communicates with the parent through the platform bridge; it is not an HTTP OpenAPI client and does not expose platform tokens or secrets to the game.

## Typical Setup

```ts
import { connectHostedGame, HostedGameError } from '@funloom/sdk/hosted';

let client;
try {
  client = await connectHostedGame({ appId: 'YOUR_FUNLOOM_APP_ID' });
  const runtime = client.runtime;
  const user = await client.user.get();
  const save = await client.save.load();
  // Initialize the game with user and save data.
} catch (error) {
  if (error instanceof HostedGameError) {
    // Present a recoverable loading/error state using error.code and error.retryable.
  }
}
```

Use `configuration.get()` to read public configured scene/item definitions, `items.list/exchange/grant/consume()` for enabled item capabilities, `ai.chat()` for configured scenes, `wallet.balance()` for balance, and `recharge.open()` to show the host recharge UI. Platform configuration and the current session determine which methods are allowed; method availability in TypeScript does not mean the capability is enabled.

## Hosted AI Scenes

- Enable AI for the game and configure one stable `sceneCode` for each model-routing policy the game needs. Multiple scene codes may point to the same model, while different scene codes may point to different platform-approved models.
- Keep system prompts and gameplay context in game code. The platform settings page stores only the scene ID, creator-facing name, enabled state, selected model, and output limit.
- Pass the game's `system`, `user`, and `assistant` messages to the Hosted bridge:

```ts
const result = await client.ai.chat({
  sceneCode: 'npc_dialogue',
  messages: [
    { role: 'system', content: 'You are the tavern keeper. Stay in character.' },
    { role: 'user', content: playerMessage },
  ],
  requestId: `npc-dialogue-${turnId}`,
});
```

- Funloom selects the configured model, measures the provider usage, and charges the current player by actual usage. Do not pass a model ID, provider, or charge amount from game code.
- A failed, incomplete, canceled, or unrecoverable call must show a retryable state and must not be treated as a successful paid result. Use the stable request ID only when retrying the same payload.

## Configured Items And Game Costs

- Read `configuration.get()` during Hosted initialization and use its public item definitions (`itemId`, `name`, `price`, `quantity`) as the source of truth. Do not hardcode an item's platform ID, display name, exchange price, or exchange-bundle quantity in game code.
- Use the configured `itemId` for `items.list`, `items.exchange`, `items.grant`, and `items.consume`; use the configured `name` in all player-facing copy, including loading, balance, success, insufficient-item, and retry states.
- `price` is the KK price of one exchange bundle and `quantity` is the number of individual item units in that bundle. `items.exchange(..., { quantity })` takes bundle count, while inventory and `items.consume(..., { quantity })` use individual units.
- Item and AI results express KK `chargedUnits` in minor units: divide by `100` before displaying it as KK币 (`4` means `0.04 KK币`). Creator-test points remain integer points and are not divided.
- A game's gameplay cost (for example, 5 item units per round) is a separate game rule. Keep it in game configuration/code unless the product explicitly adds a distinct platform field for that cost; never infer it from the exchange bundle quantity.
- If an item is renamed or its exchange ratio changes, update player-facing copy from configuration. When inventory is below the game's cost, calculate enough bundles from the configured bundle quantity before consuming the fixed game cost.
- Treat `itemId`, `name`, `price`, and `quantity` from one `configuration.get()` response as a single offer snapshot. Display and mutations in the same workflow must use that snapshot; do not display one value while sending a hardcoded or previously cached value.
- Refresh configuration before an exchange/consume workflow when the offer may have changed since initialization. The public `@funloom/sdk@0.1.2` `HostedItemOptions` is only `{ quantity?, requestId?, signal? }`; ordinary game code must not pass `expectedPrice` or `expectedQuantity`. If a particular platform host extension supports those fields, document them as host-side fields rather than public SDK API. On `PRICE_CHANGED`, discard the stale payload, reload configuration, update the UI, and require a retry with a new request ID.
- A configured bundle quantity is not a default or suggestion. If the platform changes it from 5 to 10, the game must display 10, exchange 10 units per requested bundle, use 10 in bundle-count calculations, and leave a separate fixed gameplay cost unchanged.
- If the game needs to identify one item among multiple configured items, use an explicit stable semantic mapping when the platform exposes one. If it does not, ask which configured item serves that gameplay role; never silently select the first item. Missing, invalid, or ambiguous configuration must block the paid/consuming action with a clear recoverable configuration error.
- Treat `itemId` as an inventory identity, not a display label. Dynamic lookup removes code coupling to the ID, but changing the ID can strand inventory recorded under the previous ID; confirm migration/compatibility before advising an ID change.

## Item Activity Snapshots

If the game maintains its own activity/history view, every exchange entry should preserve what happened at transaction time. Store enough metadata to explain the charge without consulting current configuration:

- game ID and game title;
- item ID and item name;
- total item units granted;
- item units per bundle;
- bundle count;
- price per bundle;
- actual charged amount and currency;
- request ID and runtime mode.

Render KK and creator-test points activity from this snapshot first. Convert KK minor units before rendering a human-facing amount. The default player-facing detail should stay concise, for example `兑换 10 个鱼干，消耗 2 KK币`; do not append the internal bundle breakdown such as `（5 个/包 × 2 包，1 KK币/包）`. Keep the bundle quantity, bundle count, and bundle price in the transaction snapshot for audit and compatibility, but do not make them part of the default game or platform activity copy. Current game titles or item configuration may be used only as compatibility fallback for older records that lack snapshots. Never reduce a financial entry to generic text such as `Hosted 游戏道具兑换`, `道具变动`, or `xx变动` when details are known.

For saves, load the current revision and pass it to `save.write()`. Handle `synced`, `pending`, and `conflict` explicitly. A conflict must preserve both local and cloud snapshots and let the player or game policy choose what to keep before calling `save.resolveConflict()`. Prefer Hosted saves over accessing parent storage or relying on iframe-local IndexedDB/localStorage for cross-session persistence.

Use stable `requestId` values when retrying the same side-effecting operation, and never reuse an ID with a different payload. Dispose the client and unsubscribe listeners when the game view is torn down. Handle `SESSION_ENDED`, timeouts, unavailable capabilities, offline pending saves, and recoverable errors in the game UI.

When the player cancels an item exchange, the host closes the confirmation and may show a single lightweight toast. A game should treat `EXCHANGE_CANCELED` as a normal user cancellation: restore its local pending UI and avoid adding a second generic failure, retry, or recharge panel.

## Build And Test

- Bundle the SDK into the uploaded game package; a browser import map or bare npm import in an unbundled static page is not sufficient.
- The package must contain a root `index.html`, all relative assets it references, and every transitive local JavaScript module imported by those assets, within current platform upload limits. A module script can request local imports as separate URLs at runtime, so checking only HTML references is insufficient.
- `connectHostedGame()` outside a valid Funloom iframe fails with `NOT_HOSTED`. Local unit/build testing is still useful for game logic, but end-to-end Hosted verification must launch from Funloom's creator "Start Test" flow.
- Test configuration-driven item behavior with at least two bundle quantities, normally 5 and 10. For each value, verify the exchange-ratio copy, bundle-count calculation, `items.exchange` payload/result, inventory balance, `items.consume` item identity, and KK/points activity detail.
- Rebuild the Hosted SDK bundle and recreate the upload package after any source or configuration-integration change. Inspect the actual directory/ZIP that will be uploaded for stale hardcoded item IDs, names, prices, quantities, and copy; do not assume an updated source tree means the upload artifact is current.
- Verify handshake, runtime and user reads, each enabled capability, disabled-capability errors, save revision/conflict recovery, session end, and cleanup. Do not claim that a local preview proves the host bridge works.
- A successful creator test does not publish the game. Use the platform's review/publish flow separately when the creator intends to release it.

For exact method signatures, protocol envelopes, limits, and error codes, use the SDK repository's Hosted SDK API reference and package README; do not copy an endpoint-based external OpenAPI example into this flow.
