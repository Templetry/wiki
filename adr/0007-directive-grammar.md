# ADR 0007 — Minimal grammar for `tpl:` directives

**Status:** ⏳ Open · **Date:** 2026-08-01

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

*Pending. To be closed during the dry-run validation of the manifest against the 3 test templates (study §7.1).*
