# Changelog

All notable changes to the Compendium VectorStore domain packages will be documented in this file.

The format is based on [Keep a Changelog](https://keepachangelog.com/en/1.1.0/),
and this project adheres to [Semantic Versioning](https://semver.org/spec/v2.0.0.html).

## [Unreleased]

### Changed

- **Every package now targets `net9.0` and `net10.0`.** .NET 9 (STS) leaves
  support on 2026-11-10; .NET 10 is the LTS consumers are moving to, and `net9.0`
  stays for the transition only. `global.json` moves to the .NET 10 SDK (10.0.100
  or any later 10.0 feature band), which builds both targets; the CI and release
  workflows install the .NET 9 SDK beside it for its runtime, so every test runs
  once per target. On `net10.0` the `Microsoft.Extensions.*` family resolves to
  10.0.13; `net9.0` keeps the 9.0.16 versions it already shipped with, so the
  `net9.0` dependency group of each nuspec is unchanged and a consumer restoring
  on the .NET 9 SDK sees the same graph as before. `Compendium.Core` and
  `Compendium.Abstractions` stay at 1.0.5-preview.1 (their `net9.0` assets are
  usable from `net10.0`). (NET10-01, agent-spot/backlog#363)
