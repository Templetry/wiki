# ADR 0007 — Minimal grammar for `tpl:` directives

**Status:** ✅ Accepted · **Date:** 2026-08-01

## Context

Directives live inside comments (ADR-0003) with the `tpl:` prefix (consistent with the Templetry brand, ADR-0008). Every new directive is public API: the fewer, the better.

## Proposed minimum

- `tpl:var <key>` — the line contains a canonical value to substitute (a point complement to the identity map).
- `tpl:if <feature>` / `tpl:endif` — conditional block per feature.

## To decide

- `tpl:else`?
- Expressions (`and`/`or`/`not`) or plain feature keys only?
- A whole-file directive (in addition to the manifest's glob-based inclusion)?
- Do v1 features support `requires`/`conflicts` between each other, or is that postponed to v1.1?

## Decision

Closed together with the dry-run validation ([spec/validation-manifests.md](../spec/validation-manifests.md)). Normative grammar in [spec/directives.md](../spec/directives.md):

- **v1 = exactly three directives**: `tpl:if <key>` (with `!` negation), `tpl:endif`, `tpl:var <key> <literal>` (line-scoped replacement).
- **No** `tpl:else` (two blocks with `!` express it), no boolean expressions, no whole-file directives, no `requires`/`conflicts` between features — all postponed; each would be new public API.
- `if`/`endif` must be alone on their line; `var` attaches to a code line. All errors report `file:line`.
- Comment-style table by extension + special basenames; unknown extension → not scanned (safe by default).

## Consequences

- The scanner stays small and testable — it is the engine's highest-risk unit and carries the densest test suite.
- Anything the grammar cannot express must be expressed by manifest structure (feature `files`, patches), keeping logic declarative and visible.
