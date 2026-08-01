# ADR 0002 — The engine is agnostic; knowledge lives in the manifest

**Status:** Accepted · **Date:** 2026-08-01

## Context

Templetry must support multiple platforms and frameworks (Android, KMP, web, backend...). The natural temptation is to write per-ecosystem adapters (an Android plugin, a React plugin...). That is the Yeoman/Backstage path, and it is the trap: every new framework would mean new code in the core — unsustainable for a single maintainer.

## Decision

The engine knows exactly **five generic operations**:

1. Copy files (binaries untouched, permissions preserved)
2. Rename via identity map (content and paths)
3. Filter comment directives
4. Apply structured patches (JSON/YAML/TOML)
5. Run a verification command in a container

All framework-specific knowledge lives in each template's `template.yml`. The `platform`/`framework` fields are catalog tags the engine ignores.

## Consequences

- Supporting a new framework = writing a template, never touching the engine.
- The manifest becomes the project's true public API — the most expensive thing to change (versioned with `schema_version` from v1).
- Project effort grows with the number of templates, not with the engine (NFR5).
