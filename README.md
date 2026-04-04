# protocol-registry

Single source of truth for AppPack protocol specs.

## Layout

- `registry.json`: master protocol manifest for local filesystem consumers
- `schemas/`: runtime-owned shared schemas synced from `../apppack-runtime/schemas/`
- `protocols/<protocol>/`: protocol-specific artifacts such as `codama.json`, `runtime.json`, `ingest.json`, `indexed-reads.json`, and optional extras like `directory.db.json`, `seed.json`, or `compute.json`
- `action-runners/`: action runner registry plus individual runner specs

## Adding or updating a protocol

1. Create or update `protocols/<protocol>/`.
2. Add the protocol entry to `registry.json` using `/protocols/<protocol>/...` paths.
3. Keep `runtime.json`, `ingest.json`, and any other internal references pointed at registry-local paths.
4. If shared schemas changed, sync them from `../apppack-runtime/schemas/` before exporting to consumers.

## Consumer sync

- `ec-ai-wallet/scripts/sync-registry.mjs` exports a flattened synced copy into `public/idl/`
- `apppack-view-service/scripts/sync-wallet-packs.mjs` exports the same flattened copy into `idl/`

Consumer copies rewrite registry-local paths back to `/idl/...` so existing wallet and view-service checks keep working.
