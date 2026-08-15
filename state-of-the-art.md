# Templetry — State of the art

**Snapshot: 2026-08-14** · the single up-to-date picture of what exists, where it lives and what state it is in. History lives in [`journal/`](journal/); decisions in [`adr/`](adr/); research in [`study/`](study/).

## What Templetry is

Templetry generates ready-to-work repositories from a library of **compilable** multi-platform templates, creates them on any git host, and keeps them alive afterwards: generated projects can pull template improvements and adopt new **pieces** long after creation.

The engine is a pure Go library plus a CLI and an MCP server; the desktop app embeds that same library behind a native UI. All framework knowledge lives in each template's `template.yml` — the engine has only generic operations ([ADR-0002](adr/0002-knowledge-lives-in-the-manifest.md), [ADR-0003](adr/0003-templates-compile.md)).

## Released components

| Repo | Role | State |
|---|---|---|
| [engine](https://github.com/Templetry/engine) | Go library + `templetry` CLI + `templetry-mcp` server | **v1.7.0** — binaries for linux/darwin/windows (amd64 + arm64) with SHA256SUMS |
| [desktop](https://github.com/Templetry/desktop) | Native app (Wails: Go backend embedding the engine, React/TS frontend) | **v1.5.0** — Windows (installer + portable), Linux, macOS universal |
| [catalog](https://github.com/Templetry/catalog) | Default registry (`registry.json`, schema v2) | **21 forms across 10 parents**, all `ready`; **12 pieces** |
| [scoop-bucket](https://github.com/Templetry/scoop-bucket) | `scoop install templetry` (CLI + MCP), autoupdating | live |
| [wiki](https://github.com/Templetry/wiki) | Studies, ADRs, specs, journal, brand | living |
| [.github](https://github.com/Templetry/.github) | Organization profile | living |

Licences: engine under **PolyForm Noncommercial 1.0.0**; every other repo **MIT**.

## The catalog

Ten parents, every form a real project whose CI renders **and compiles** it on each push. `status: ready` in the registry is gated on green CI.

| Parent | Forms | Pieces |
|---|---|---|
| [kmp](https://github.com/Templetry/kmp) | `modular-features`, `single-module`, `modular-ui` | — |
| [android](https://github.com/Templetry/android) | `modular-features`, `single-module` | — |
| [web](https://github.com/Templetry/web) | `react-spa`, `vue-spa`, `nextjs`, `svelte-spa` | `axios-api`, `zustand-store`, `pinia-store`, `zod-env` |
| [go](https://github.com/Templetry/go) | `cli`, `http-service`, `rest-sqlite` | `version-endpoint`, `crud-resource` |
| [python](https://github.com/Templetry/python) | `fastapi-service`, `cli-typer`, `fastapi-users` | `rbac`, `api-keys`, `audit-trail`, `soft-delete`, `verifactu`, `crud-resource` |
| [rust](https://github.com/Templetry/rust) | `cli`, `axum-service` | — |
| [node](https://github.com/Templetry/node) | `express-api` | — |
| [jvm](https://github.com/Templetry/jvm) | `spring-boot` | — |
| [dotnet](https://github.com/Templetry/dotnet) | `minimal-api` | — |
| [meta](https://github.com/Templetry/meta) | `template` (the template that creates templates) | — |

**Eleven ecosystems, one engine, one manifest schema** — Kotlin Multiplatform, Android, React, Vue, Next.js, Svelte, Go, Node/TypeScript, Python, Rust, JVM/Spring, .NET. None of them required an engine change: the strongest evidence for the agnostic-engine thesis (ADR-0002).

## Shipped capabilities

### Engine + CLI (v1.7.0)

- **Render**: manifest parsing/validation, derived casings, feature exclusion, identity renaming, comment directives, structured patches for `.json`/`.yml`/`.toml`, deterministic output (same inputs → byte-identical result).
- **`requires`/`conflicts` between features and `presets`** (named feature combos, resolved as defaults → preset → explicit).
- **Remote catalogs and multi-forge hosting**: `github:`, `gitlab:` and `gitea:` source schemes (one adapter covers Gitea, Forgejo, Codeberg and self-hosted), with legacy bare refs untouched ([ADR-0015](adr/0015-multi-forge-foundation.md)).
- **`templetry update`**: re-render at the template's head with the recorded inputs, diff against disk, three-way merge via `git merge-file`; adds and modifications only, never deletes.
- **`templetry verify`**: runs the manifest's `verify: {image, run}` in Docker (ADR-0004 realized).
- **Pieces** (`templetry pieces` / `add`): decoupled units adopted after creation, with their own variables, identity map and drift anchor ([ADR-0014](adr/0014-lazy-pieces.md)).
- **Hardened**: output paths cannot escape or abuse platform semantics (Windows device names, case-insensitive collisions, trailing dot/space); directive scanner fuzz-tested; manifests tolerate a UTF-8 BOM.
- **`templetry-mcp`**: dependency-free MCP server exposing `list_templates`, `get_form_schema`, `plan`, `render`, `update`, `list_pieces`, `add_piece`.

### Desktop (v1.5.0)

Three sections:

- **Build** — pick a form, fill the manifest-driven dynamic form (with preset selector), preview the render, create the repo and push.
- **Cloud** — repositories across every signed-in account: GitHub (OAuth device flow), GitLab and Gitea/Forgejo (personal access token in the OS keyring). Engine-readable template repos are flagged, local clones cross-linked, and each repo opens a state preview: languages, branches, latest CI runs, README and docs rendered as sanitized markdown.
- **Local** — recursive scan of the repositories folder organized by on-disk folders; per-repo preview with branches, remotes, last commit and working-tree state; drift detection and the assisted update cycle.

Plus **BYOR**: paste any empty repository's URL and Templetry renders, commits and pushes — every git host on earth, no adapter needed. Ships for Windows, Linux and macOS; the in-app updater is Windows-only for now.

### Pieces: the industrial catalog

Researched in [study VIII](study/industrial-pieces-v1.md) with one rule — **encode a standard, not an opinion**:

| Piece | Encodes |
|---|---|
| `rbac` | NIST RBAC (ANSI/INCITS 359-2004): users → roles → permissions `(object, operation)`, with route guards |
| `api-keys` | Hashed keys with indexed prefix lookup, scopes, expiry, revocation, plaintext shown once |
| `verifactu` 🇪🇸 | Spain's RD 1007/2023: SHA-256 fingerprint chain, append-only registry, event log, QR payload ([study IX](study/verifactu-v1.md)) |
| `audit-trail` | Append-only who/what/when/where, with no write routes at all |
| `soft-delete` | `deleted_at` with opt-out filtering, restore, explicit purge |
| `crud-resource` | A whole entity (model, repository/router, tests) renamed to your object |

Pieces wire themselves through **sockets** — a registration point the piece plugs into from a file it owns — so adopting one never edits an existing file. Proven in Go (`api.Register`, `store.Register`) and Python (routers package walk).

## Decisions in force

Fifteen ADRs, indexed in [`adr/README.md`](adr/README.md). Load-bearing ones: own engine in Go (0001, 0006), agnostic engine (0002), templates compile (0003), verify in containers (0004), updates prepared from day one (0005), minimal directive grammar (0007), multi-forge (0009 + 0015), parents/forms/features (0011), desktop with Wails (0012 ⊃ 0010), **v1.0.0 declared** (0013), **lazy pieces** (0014).

Discarded along the way: the hosted web app (superseded by the desktop).

## Specs

Normative documents in [`spec/`](spec/): [`template-yml.md`](spec/template-yml.md) (the manifest), [`directives.md`](spec/directives.md), [`answers-file.md`](spec/answers-file.md), [`piece-yml.md`](spec/piece-yml.md), [`validation-manifests.md`](spec/validation-manifests.md), and [`compatibility.md`](spec/compatibility.md) — **in force since v1.0.0**: the manifest, directives, answers file, registry v2, CLI surface and exported Go API only break with a major.

## Where next

A proposed plan lives in [study X](study/where-next-v1.md): keep the promises already made (piece updates, pieces in the desktop), **stop the rot** (scheduled CI in every parent — nothing is verified on a schedule today, and `kmp`/`android` still render with engine v0.1.0), pay the waived soak window by dogfooding real projects, and only then decide whether adoption is a goal.

## Open backlog

- **Pieces** (study VIII): `scim-provisioning` (RFC 7643), `oidc-login`, `multi-tenancy`, `outbox`, `rate-limit`, integrations (`stripe-billing`, `s3-storage`), domain CRUDs.
- **Engine**: `requires`/`conflicts` **between pieces** and a jurisdiction/compat axis for pieces like `verifactu` — both surfaced by study VIII.
- **Desktop**: pieces panel (list/add from the UI), piece drift in the update cycle, settings sync, code signing, in-app update on macOS/Linux.
- **Distribution**: winget (after signing), Homebrew tap.
- **Forge adapters**: Bitbucket if anyone asks; BYOR already covers it.
