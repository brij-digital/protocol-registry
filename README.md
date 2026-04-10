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

## Canonical Model

`registry.json` currently carries two main top-level catalogs:

- `protocols[]`
  - canonical protocol metadata
  - runtime pack location via `agentRuntimePath`
  - codama location via `codamaIdlPath`
- `indexings[]`
  - canonical indexing definitions
  - source list via `sources[]`
  - entity projection directory via `entitySchemaPath`

For indexing, the canonical path is now:
- `indexings[].sources[].ingestSpecPath`
- `indexings[].entitySchemaPath`

`entitySchemaPath` now points directly to `indexing/entities/<indexingId>/`.
Consumers scan that directory and merge one file per entity.

## Current Indexings

The active canonical indexings today are:
- `orca-whirlpool-mainnet`
- `pump-amm-mainnet`
- `pump-core-mainnet`

Their specs live under:
- `indexing/ingest/`
- `indexing/entities/`

Entity authoring layout:
- `indexing/entities/<indexingId>/<EntityName>.json`

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
   - entity shards under `indexing/entities/`
4. Register the protocol and/or indexing in `registry.json`.
5. Prove the change in `protocol-conformance` when relevant.

## Notes

- This repo should stay boring and lightweight.
- Do not commit `node_modules`, generated artifacts, or local tooling state here.
- If spec validation tooling is needed, it should live in a consumer repo or CI, not by turning this repo back into a pseudo-application.
