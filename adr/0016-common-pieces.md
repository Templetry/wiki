# ADR 0016 — Common pieces: one idea, many implementations

**Status:** ✅ Accepted · **Date:** 2026-08-15 · Realizes phase 2 of [study VII](../study/pieces-v1.md)

## Context

Pieces live inside the form that ships them ([ADR-0014](0014-lazy-pieces.md)). That was the right v1: it kept discovery trivial (the form tarball carries everything) and needed no registry change.

It stops scaling the moment an idea is not form-specific. `audit-trail`, `renovate`, `editorconfig`, `codeowners`: the *concept* is identical across the catalog's eleven ecosystems, only the *implementation* differs — and some differ not at all. Under the v1 model, `audit-trail` would be copied into every form that wants it, and every fix would have to be copied too. That is the maintenance pit ADR-0009 taught us to avoid.

## Decision

**A piece may live outside any form, in a shared repository, and declare what it applies to.**

1. **`piece.yml` gains `applies_to`**: a list of template *names* (the `name` field of `template.yml`, already recorded in every answers file). Empty means universal.
2. **Registry v2.1 gains a top-level `pieces` list** — additive, `schema_version` stays 2: each entry carries `name`, `repo`/`source`, `ref` and `path`. The registry is fetched anyway, so discovery costs nothing extra.
3. **Same name, different implementations**: two directories may declare the same piece `name` with disjoint `applies_to`. The one whose `applies_to` matches the project wins; a project therefore asks for `audit-trail`, not for `audit-trail-fastapi`.
4. **Form-local pieces win over common ones** on a name clash: a form that ships its own implementation is making a deliberate statement.
5. **Updates follow the recorded source.** The answers file already stores each applied piece's `source` and `commit`; the update cycle re-renders a piece from *its own* source rather than assuming it lives in the form. This is what makes a common piece maintainable — fix it once, every project that adopted it sees the fix.

## Consequences

- The catalog gains a second axis: forms scale by ecosystem, pieces scale by concern.
- A universal piece (`renovate`, `editorconfig`) is written once and applies to all 21 forms; a per-ecosystem one is written once per ecosystem instead of once per form.
- All changes are additive: existing form-local pieces, answers files and registries keep working untouched, and the compatibility policy holds.
- Cost accepted: a project's pieces may now come from two repositories, so `templetry pieces` performs one extra fetch. Worth it — the alternative is copies that drift.
