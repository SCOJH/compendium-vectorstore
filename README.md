# Compendium — VectorStore Domain

Domain repository for the Compendium vector-store surface (repo-per-domain topology, ADR-0007):
the `IVectorStore` abstraction and every adapter that implements it live here together, so a
change to the port and its implementations ships as **one atomic PR** and **one version train**.

## Packages

| Package | Project | Description |
|---|---|---|
| `Compendium.Abstractions.VectorStore` | `src/Compendium.Abstractions.VectorStore` | The `IVectorStore` port: embedding storage, similarity search, tenant-aware filtering. Provider-agnostic models (`VectorRecord`, `VectorMatch`, `VectorFilter`, `DistanceMetric`) and `VectorStoreErrors`. |
| `Compendium.Adapters.Pgvector` | `src/Compendium.Adapters.Pgvector` | PostgreSQL + pgvector implementation: per-collection tables with HNSW/IVFFlat indexes, tenant-isolated upsert/search/delete, JSONB metadata, Result-pattern error handling. |
| `Compendium.Adapters.Pinecone` | `src/Compendium.Adapters.Pinecone` | Pinecone adapter (scaffold — vendor implementation pending). |
| `Compendium.Adapters.Qdrant` | `src/Compendium.Adapters.Qdrant` | Qdrant adapter (scaffold — vendor implementation pending). |

All four packages version together from git tags (MinVer, tag prefix `v`). The domain train is
`1.1.x` — deliberately above the framework's `1.0.x` so domain-repo packages win resolution over
the legacy framework-published `Compendium.Abstractions.VectorStore`.

## Layout

```
src/
  Compendium.Abstractions.VectorStore/   # the port (packable)
  Compendium.Adapters.Pgvector/          # adapters reference it via ProjectReference
  Compendium.Adapters.Pinecone/
  Compendium.Adapters.Qdrant/
tests/
  Unit/                                  # one unit test project per src project
  Integration/                           # [RequiresDocker]-gated (Testcontainers)
```

Compendium base packages (`Compendium.Core`, `Compendium.Abstractions`) are consumed from
nuget.org and pinned centrally in `Directory.Packages.props`.

## Build & test

```bash
dotnet build -c Release
dotnet test  -c Release --filter "FullyQualifiedName!~IntegrationTests"   # unit
dotnet test  -c Release   # + integration (requires Docker)
```

- .NET 9 / C# 13, `TreatWarningsAsErrors`, central package management.
- CI enforces a 65% line-coverage gate on the unit-testable surface (measured 67.7% at
  assembly; the pgvector raw driver is integration-covered).

## Release

Push a tag `vX.Y.Z[-preview.N]` → the Release workflow packs the 4 packages and publishes to
GitHub Packages (primary feed); nuget.org publish runs when `NUGET_API_KEY` is configured and
soft-skips otherwise.

## Provenance

This repo was assembled from the framework repo and three standalone adapter repos —
see [MIGRATION.md](MIGRATION.md) for exact source SHAs and adaptation notes.
