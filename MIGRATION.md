# MIGRATION — vectorstore domain repo assembly

This repository was assembled on 2026-07-25 from the Compendium framework repo and the three
standalone vector-store adapter repos (repo-per-domain topology, ADR-0007). Sources were
extracted read-only via `git archive` at the SHAs below; the source repos were not modified.

## Component provenance

| Component | Source repo | Source ref | Source HEAD SHA | Notes |
|---|---|---|---|---|
| `src/Compendium.Abstractions.VectorStore` | `sassy-solutions/compendium` (framework) | `origin/main` | `792dd626496f69a4f7c86ce79f48795e6aba7e34` | Was `src/Abstractions/Compendium.Abstractions.VectorStore`. `ProjectReference` to `Compendium.Core` replaced by a nuget.org `PackageReference` (pinned `1.0.5-preview.1`); `IsPackable=true` + `PackageTags` added explicitly (the framework's root props supplied packability there). PackageId unchanged. |
| `tests/Unit/Compendium.Abstractions.VectorStore.Tests` | `sassy-solutions/compendium` (framework) | `origin/main` | `792dd626496f69a4f7c86ce79f48795e6aba7e34` | Was `tests/Unit/Compendium.Abstractions.VectorStore.Tests`. ProjectReference path re-rooted to the flattened `src/` layout. |
| `src/Compendium.Adapters.Pgvector` + unit/integration tests | `sassy-solutions/compendium-adapter-pgvector` | `main` | `afeac6800817068a3b4cce95125c18042c62d600` | `PackageReference Compendium.Abstractions.VectorStore` (was pin `1.0.1`) replaced by in-repo `ProjectReference` in src + both test projects. `samples/01-rag-roundtrip` intentionally not migrated. |
| `src/Compendium.Adapters.Pinecone` + unit tests | `sassy-solutions/compendium-adapter-pinecone` | `main` | `90d16c27693af7a3dedd33694cf9a126e32d2be6` | Scaffold placeholder (Echo adapter; does not implement `IVectorStore` yet). `ProjectReference` to the in-repo abstraction ADDED (it had none) so the vendor implementation lands as an atomic PR. |
| `src/Compendium.Adapters.Qdrant` + unit tests | `sassy-solutions/compendium-adapter-qdrant` | `main` | `15d4c4b25a24d23f458460ff1f06b03dd15444f7` | Same as Pinecone: scaffold placeholder; abstraction `ProjectReference` added. |
| Scaffold (`Directory.Build.props`, `global.json`, `.gitignore`, `.config/dotnet-tools.json`, `LICENSE`, workflow templates) | `sassy-solutions/compendium-adapter-supabase` | working tree `HEAD` | `86bad144bb57ed1284e17d88bb2acfe45d954692` | Freshest release.yml (GitHub Packages first, nuget.org soft-skip, multi-nupkg assert). URLs re-pointed to `SCOJH/vectorstore`; feed owner parameterized via `github.repository_owner`; nupkg assert adapted to `-ge 4`. |

## Central package versions

`Directory.Packages.props` is the union of the source repos' pins, conflicts resolved to the
highest version:

- `Compendium.Core` / `Compendium.Abstractions`: `1.0.5-preview.1` (nuget.org; sources pinned `1.0.1` / `1.0.0-preview.8`)
- `Compendium.Abstractions.VectorStore`: **removed from pins** — now built in-repo
- `Microsoft.Extensions.*`: `9.0.16` (pgvector) over `9.0.0` (pinecone/qdrant)
- Test stack, Npgsql `9.0.4`, Pgvector `0.3.2`, Testcontainers `4.11.0`: identical across sources

## Versioning

- MinVer, tag prefix `v`. First tag: `v1.1.0-preview.1` — the domain train sits ABOVE the
  framework's `1.0.x` so domain-repo packages win resolution.

## Verification at assembly time

- `dotnet build -c Release`: 0 errors, 0 warnings (no API drift — pgvector compiled unchanged
  against the origin/main abstraction and `Compendium.Core 1.0.5-preview.1`).
- Unit tests: 226/226 passed (49 abstraction, 143 pgvector, 17 pinecone, 17 qdrant).
- Integration tests: `[RequiresDocker]`-gated; Docker unavailable in the assembly environment,
  not executed.
- Line coverage: 67.7% (abstraction 98.8%, pgvector 60.6%, pinecone/qdrant 100%); CI gate set
  to 65%.

## Follow-ups (separate steps, NOT done here)

- Remove `Compendium.Abstractions.VectorStore` from the framework repo.
- Archive `compendium-adapter-pgvector`, `compendium-adapter-pinecone`,
  `compendium-adapter-qdrant`.
- Implement the real Pinecone and Qdrant adapters against `IVectorStore`.
