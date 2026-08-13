# 2026-08-13 — Catalog expansion: five ecosystems in one pass

The catalog grew from 4 parents / 8 forms to **9 parents / 14 forms**: `go` (cli, http-service), `node` (express-api), `python` (fastapi-service), `jvm` (spring-boot) and `dotnet` (minimal-api) — every form a compilable project with AGENTS.md, a declared `verify:` block, optional Dockerfile and a Verify CI that renders and builds with the real toolchain.

## What the pass proved

- **The engine never moved.** Five new ecosystems, one manifest schema, zero engine changes — the ADR-0002 thesis under its widest test yet. Python needed `tpl:var` in `pyproject.toml`; the rest was identity map and feature globs.
- **The dry-run manifests came true.** Study I hand-wrote a FastAPI manifest as a schema stress test in Phase 0; it shipped almost verbatim, `python_version` select included.
- **Local verification where the toolchain existed**: go and node forms (and their renders) were built and tested on the author's machine before pushing; python/jvm/dotnet were validated for identity and structure locally, and compiled by their CI.

## The one red

`dotnet/minimal-api` failed its first CI: implicit usings do not cover xUnit, so the rendered test project missed `using Xunit;`. One-line fix, green on rerun. Same lesson as the 13-round Kotlin sanitation of 2026-08-02, at 1/13 the cost: **the defect surfaced in the template's own CI, not in a user's project** — and it surfaced *because* the CI compiles the render rather than trusting the template.

## Second pass, same day

A follow-up round added the `rust` parent (`cli` with clap, `axum-service` with axum + tokio), two more web forms (`vue-spa`, `nextjs`) and `python/cli-typer` — **10 parents / 18 forms across eleven ecosystems**, all green. Notes:

- The two SPAs (`react-spa`, `vue-spa`) switch their entry point with the *same* HTML-comment directive pattern — one comment-style table, two frameworks.
- Rust needed two casings of one identity (`verify-app` package, `verify_app` import): pure identity-map work, no engine change.
- Parent CIs graduated to matrices (form × preset, form × python version), so new forms cost a matrix entry rather than a workflow.

## Takeaway

Adding an ecosystem is now a bounded, repeatable job: write a compilable form, declare its identity, ship AGENTS.md, wire a Verify job, flip the registry when green. The cost curve grows with templates, not with the engine (NFR5), exactly as designed.
