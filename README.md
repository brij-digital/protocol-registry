# Protocol Registry

Source of truth for all AppPack protocol specs.

## Structure

```
registry.json              # Master registry — lists all protocols and paths
schemas/                   # Shared JSON schemas (owned by apppack-runtime)
runtime/                   # Agent runtime specs (writes, views, transforms)
codama/                    # Codama IDLs (on-chain program descriptions)
indexing/
  ingest/                  # Ingest specs (Carbon pipeline definitions)
  indexed-reads/           # Indexed read specs (query projections)
action-runners/            # Multi-step action runner specs
```

## Protocols

| Protocol | Swap | LP | Quotes | Indexing |
|---|---|---|---|---|
| **Orca Whirlpool** | ✅ swap, two-hop | ✅ open, close, increase, decrease | ✅ exact-in, exact-out, LP quotes | ✅ trades, snapshots |
| **Pump AMM** | ✅ buy, sell | — | ✅ preview buy/sell | ✅ trades, snapshots |
| **Pump Core** | ✅ buy | — | ✅ preview buy | ✅ trades, curves |

## How It Works

- **Agents** load protocol packs from this registry to execute operations locally
- **View service** syncs indexing specs for Carbon ingestion and query projections
- **Wallet** syncs specs for the browser UI
- **Conformance tests** validate spec correctness against official SDKs

## Adding a Protocol

1. Add Codama IDL to `codama/`
2. Author runtime spec in `runtime/`
3. Author ingest + indexed-reads specs in `indexing/`
4. Add entry to `registry.json`
5. Add conformance tests to `protocol-conformance` repo

## Consumers

All repos sync from this registry:
- `ec-ai-wallet` — syncs to `public/idl/`
- `apppack-view-service` — syncs to `idl/`
- `protocol-conformance` — reads directly via registry path
