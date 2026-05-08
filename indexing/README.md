# Indexing Specs

This directory defines the canonical indexing specs consumed by
`protocol-indexing`.

## Model

- `ingest/` defines source ingestion into canonical raw/state tables.
- `entities/` defines API datasets.
- Each API dataset is its own `index_id`.
- `index_id` is the unit of retention, limits, health, metrics, and pricing.
- Do not group entity specs by protocol directory.

## Registry Wiring

`registry.json#indexings[]` owns source ingestion:

```json
{
  "id": "pump-amm-mainnet",
  "sources": [
    {
      "id": "pump-amm",
      "protocolId": "pump-amm-mainnet",
      "ingestSpecPath": "/indexing/ingest/pump-amm.json"
    }
  ]
}
```

`registry.json#indexes[]` owns API datasets:

```json
{
  "id": "pump-amm-trade-mainnet",
  "sourceIndexingIds": ["pump-amm-mainnet"],
  "entitySchemaPath": "/indexing/entities/pump-amm-trade-mainnet.json",
  "status": "active"
}
```

## Entity Spec Contract

Each file in `entities/` must be a flat entity spec:

```text
indexing/entities/<index_id>.json
```

Required fields:

- `indexId`: must match `registry.json#indexes[].id`.
- `entityName`: API entity name.
- `sourceIndexingIds`: source dependencies.
- `retention`: projection retention window.
- `limits`: operational limits.
- `source`: raw/state source binding.
- `transform`: extraction, compute, resolve, and reduce rules.
- `emit`: API id, fields, indexes, and default ordering.

Minimum `limits` shape:

```json
{
  "maxLagSeconds": 30,
  "maxBacklogRows": 1000000,
  "maxTableBytes": 107374182400,
  "maxApiP95Ms": 1000
}
```

The active schema is:

```text
indexing/entities/entity_definition_schema.v1.json
```
