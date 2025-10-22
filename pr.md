Bugfix: stabilize UI utils with provider/indexer fallbacks

Problem
- `provider.api().getLastBlock` is missing for some providers (e.g. TON Connect via blueprint run), causing:
  `TypeError: provider.api(...).getLastBlock is not a function`
- `waitForTransaction` was tightly coupled to lite API calls.

Repro
- `npx blueprint run` → `minterController` → `Mint` (on testnet via TonConnect wallet)
- Crash in `wrappers/ui-utils.ts: getLastBlock`

Fix
- getLastBlock: if provider has no `getLastBlock` → fallback to Toncenter indexer `block/latest`.
- getAccountLastTx: layered attempts
  1) `getAccountLite(lastBlock, address)` when available,
  2) `getTransactions(address, { limit: 1 })` when available,
  3) Toncenter v3 indexer `account` endpoint.
- waitForTransaction: compare by `lt` via `getAccountLastTx` without direct lite-API dependency.
- sendToIndex: normalize params (Address → string, etc.) to avoid `[object Object]`.
- wallet endpoint call: `address: address.toString()`.

Impact
- Scripts in `scripts/` such as `minterController` work again with TonConnect/blueprint without runtime errors.
- Backward-compatible behavior: if provider exposes native `getLastBlock`/`getAccountLite`, they are used.

Files
- `wrappers/ui-utils.ts`

Notes
- No public API break; fallbacks are internal.

