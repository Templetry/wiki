# ADR 0015 — Multi-forge foundation: BYOR and source schemes

**Status:** ✅ Accepted · **Date:** 2026-08-13 · Revives [study V](../study/multi-forge-hosting-v1.md), refines [ADR-0009](0009-multi-forge.md)

## Context

ADR-0009 accepted multi-forge with a universal fallback — "bring your own remote" (BYOR): the user creates an empty repo anywhere and pastes its URL; the app only pushes. Study V was archived on the grounds that BYOR covered exotic hosts.

**Audit finding (2026-08-13): BYOR was never implemented.** The desktop offers a GitHub owner or local-only; there is no place to paste a remote. The v1 claim "BYOR covers every host" describes a design, not shipped code. Meanwhile the engine's template fetching is hardwired to `codeload.github.com`, so third-party catalogs cannot live on GitLab or Gitea either.

## Decision

Split multi-forge into its two independent axes and build the foundation of both, without forge adapters yet:

1. **Read axis — source schemes in the engine.** `source.Fetch(ref, subdir, token)` dispatches on a scheme prefix: `github:owner/repo` (bare refs default to it, preserving every existing answers file and registry), `gitlab:host/group/project`, `gitea:host/owner/repo` (one adapter serves Gitea, Forgejo, Codeberg and self-hosted instances). Each backend only builds a tarball URL and resolves a ref to a commit; `FromTarGz` is already generic. Registry gains an optional per-parent `source` field (schema v2.1, additive).
2. **Write axis — BYOR, implemented.** The desktop's owner picker gains a "paste a remote URL" option: the app renders locally, initializes the repo, commits and pushes to the given URL. No API, no adapter, works against every git host on earth.

Forge adapters (GitLab, Gitea) stay deferred: with BYOR real, an adapter buys convenience (creating the repo for you, listing your groups), not capability. They land when a user asks — the same anti-maintenance-pit rule ADR-0009 set.

## Consequences

- The multi-forge claim becomes true in code, not just in an ADR.
- Templates and catalogs can be hosted outside GitHub; the answers file records the full scheme, so the update cycle follows the template wherever it lives.
- All additive: bare refs keep working, so every existing project keeps updating unchanged.
- Auth stays per-host (tokens passed in by the caller); the desktop keeps GitHub OAuth for identity and will add per-host tokens when adapters arrive.
