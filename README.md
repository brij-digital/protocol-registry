# Protocol Registry

Canonical spec registry for AppPACK.

This repo is intentionally a data/spec repository:
- no runtime service
- no build system
- no npm toolchain
- no generated dependency artifacts

It stores the protocol metadata and JSON specs consumed by the other repos in this workspace.

## What Lives Here

- `registry.json`: canonical top-level registry
- `schemas/`: shared JSON schemas
- `runtime/`: Solana agent runtime specs for reads, writes, and reusable transforms
- `codama/`: codama/IDL source files
- `indexing/ingest/`: canonical ingest specs
- `indexing/entities/`: canonical entity projection specs for `protocol-indexing`
- `action-runners/`: action-runner registry for higher-level linear flows
- `action-policy-templates/`: canonical policy templates for action runners

## Canonical Model

`registry.json` carries three top-level catalogs:

- `protocols[]`
  - canonical protocol metadata
  - runtime pack location via `agentRuntimePath`
  - codama location via `codamaIdlPath`
- `indexings[]`
  - canonical source-ingest definitions
  - source list via `sources[]`
- `indexes[]`
  - canonical API datasets
  - one entry per `index_id`
  - source dependencies via `sourceIndexingIds`
  - entity projection spec location via `entitySchemaPath`

For indexing, the canonical path is now:
- `indexings[].sources[].ingestSpecPath`
- `indexes[].entitySchemaPath`

`indexings[]` does not own API datasets. It only defines how source data enters
the canonical raw/state layer.

`indexes[]` owns API datasets. `indexes[].entitySchemaPath` points directly to
one flat entity spec file:

```text
indexing/entities/<index_id>.json
```

Do not group entity specs by protocol directory. A protocol can have many API
datasets, and each dataset is administered by its own `index_id`.

Each entity spec must declare:
- `indexId`: exactly the same id as `indexes[].id`
- `sourceIndexingIds`: source dependencies
- `retention`: projection retention window
- `limits`: operational limits for lag, backlog, table bytes, and API P95
- `source`, `transform`, and `emit`

The active entity schema is:

```text
indexing/entities/entity_definition_schema.v1.json
```

## Current Indexings

The active canonical indexings today are:
- `orca-whirlpool-mainnet`
- `pump-amm-mainnet`
- `pump-core-mainnet`

Their specs live under:
- `indexing/ingest/`
- `indexing/entities/`

The active API indexes today are:
- `orca-whirlpool-pool-mainnet`
- `orca-whirlpool-pool-volume-1m-mainnet`
- `orca-whirlpool-position-mainnet`
- `orca-whirlpool-swap-mainnet`
- `pump-amm-pool-mainnet`
- `pump-amm-trade-mainnet`
- `pump-amm-trade-volume-1m-mainnet`
- `pump-core-bonding-curve-account-mainnet`

Entity authoring layout is flat:
- `indexing/entities/<index_id>.json`

## Action Policy Templates

Action runners are global protocol capabilities. Action policy templates describe the
policy controls and safety checks that can be attached to those runners by an app,
agent, session, or user.

This repo stores only templates. Concrete user policies and control values should live
in the consuming app, MCP server, wallet layer, or database.

Template layout:
- `action-policy-templates/action_policy_templates.json`
- `action-policy-templates/<actionId>.template.json`

## Consumers

This repo is consumed by:
- `protocol-runtime` for runtime packs and schema-driven execution
- `protocol-indexing` for ingest specs and entity projection specs
- `protocol-ui` for synced public protocol artifacts
- `protocol-conformance` for parity and registry-backed conformance tests

Validation and execution happen in those consuming repos, not here.

## Adding Or Updating Specs

Typical flow:

1. Add or update the codama file under `codama/`.
2. Add or update the runtime pack under `runtime/`.
3. If the protocol is indexed, add or update:
   - ingest spec under `indexing/ingest/`
   - entity spec under `indexing/entities/<index_id>.json`
4. Register source ingest in `registry.json#indexings[]`.
5. Register each API dataset in `registry.json#indexes[]`.
6. If the action is exposed through a runner, add or update the runner under `action-runners/`.
7. If the action needs agent safety controls, add or update its template under `action-policy-templates/`.
8. Prove the change in `protocol-indexing` and `protocol-conformance` when relevant.

## Notes

- This repo should stay boring and lightweight.
- Do not commit `node_modules`, generated artifacts, or local tooling state here.
- If spec validation tooling is needed, it should live in a consumer repo or CI, not by turning this repo back into a pseudo-application.
