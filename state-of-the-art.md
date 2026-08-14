# Templetry — State of the art

**Snapshot: 2026-08-07** · maintained as the single up-to-date picture of what exists, where it lives and what state it is in. History lives in [`journal/`](journal/); decisions in [`adr/`](adr/).

## What Templetry is

Templetry generates ready-to-work repositories from a library of **compilable** multi-platform templates and creates them on any git host. The engine is a pure Go library + CLI; the desktop app embeds that library behind a native UI. All framework knowledge lives in each template's `template.yml` manifest — the engine has only generic operations ([ADR-0002](adr/0002-knowledge-lives-in-the-manifest.md), [ADR-0003](adr/0003-templates-compile.md)).

## Where it lives

Everything is published in the GitHub organization **[Templetry](https://github.com/Templetry)** (migrated from personal repos in August 2026; the pre-org repos `android-native-base`, `kmp-native-base` and `compose-multiplatform-app` were folded into the parents and deleted).

| Repo | Role | State |
|---|---|---|
| [engine](https://github.com/Templetry/engine) | Pure Go library + `templetry` CLI (`list`, `init`, `plan`, `render`, `verify`, `update`, `pieces`, `add`, `version`) + `templetry-mcp` MCP server | **v1.5.0** — multi-forge template hosting (`github:`/`gitlab:`/`gitea:` schemes); public API under the [compatibility policy](spec/compatibility.md); binaries for linux/darwin/windows (amd64 + arm64) |
| [desktop](https://github.com/Templetry/desktop) | Native desktop app (Wails: Go backend embedding the engine, React/TS frontend) | **v1.5.0** — Windows, Linux and macOS builds; GitHub/GitLab/Gitea accounts + BYOR on any git host |
| [catalog](https://github.com/Templetry/catalog) | Default template registry — `registry.json` (schema v2) consumed by CLI and desktop | 19 forms across 10 parents, all `ready`; 5 lazy pieces |
| [kmp](https://github.com/Templetry/kmp) | Parent: Kotlin Multiplatform | forms `modular-features`, `single-module`, `modular-ui` — all ready, CI-verified |
| [android](https://github.com/Templetry/android) | Parent: Android native | forms `modular-features`, `single-module` — both ready, CI-verified |
| [web](https://github.com/Templetry/web) | Parent: Web | forms `react-spa`, `vue-spa`, `nextjs`, `svelte-spa` — ready, CI-verified on both presets; pieces `axios-api`, `zustand-store`, `pinia-store`, `zod-env` |
| [rust](https://github.com/Templetry/rust) | Parent: Rust | forms `cli` (clap), `axum-service` (axum + tokio) — ready, CI-verified |
| [go](https://github.com/Templetry/go) | Parent: Go | forms `cli`, `http-service` (stdlib only, distroless Docker) — ready, CI-verified; piece `version-endpoint` via the api socket |
| [node](https://github.com/Templetry/node) | Parent: Node.js | form `express-api` (Express 5 + TS NodeNext, Vitest) — ready, CI-verified |
| [python](https://github.com/Templetry/python) | Parent: Python | forms `fastapi-service`, `cli-typer` — ready, CI-verified on 3.12 and 3.13 |
| [jvm](https://github.com/Templetry/jvm) | Parent: JVM | form `spring-boot` (Kotlin, Gradle KTS, RANDOM_PORT tests) — ready, CI-verified |
| [dotnet](https://github.com/Templetry/dotnet) | Parent: .NET | form `minimal-api` (C# top-level, xUnit + WebApplicationFactory) — ready, CI-verified |
| [meta](https://github.com/Templetry/meta) | Parent: the template that creates Templetry templates | form `template` ready |
| [wiki](https://github.com/Templetry/wiki) | This repo: studies, ADRs, specs, journal, brand | living |
| [scoop-bucket](https://github.com/Templetry/scoop-bucket) | Windows package manifest: `scoop install templetry` (CLI + MCP), autoupdating | live |
| [.github](https://github.com/Templetry/.github) | Organization profile | living |

Licenses: engine under **PolyForm Noncommercial 1.0.0**; every other repo **MIT**.

Local layout mirrors the org under `Repos\templetry\` with the parents grouped in `parents\`.

## Shipped capabilities

### Engine + CLI (v1.0.0)

- Manifest parsing + validation + derived casings; planner (feature exclusion, identity renaming); renderer (directive scanner, JSON patches); deterministic output — same inputs, byte-identical result.
- Remote catalogs: `templetry list` / `templetry init parent/form` resolve against the official registry (`https://raw.githubusercontent.com/Templetry/catalog/main/registry.json`) or any custom one.
- `templetry verify`: renders to a temp dir (or takes `--dir`) and runs the manifest's `verify: {image, run}` in Docker (ADR-0004 fully realized).
- Hardened output paths: no escapes, absolute paths, Windows device names, trailing dot/space segments or case-insensitive dest collisions; directive scanner fuzz-tested; manifests tolerate a UTF-8 BOM.
- Answers file records every input **and the resolved template commit** (drift anchor) — the foundation for the update cycle ([ADR-0005](adr/0005-prepare-for-updates.md)).
- Releases with SHA256SUMS; CI on every push.

### Desktop (v1.0.0)

Three main sections:

- **Build** — repo-first creation: pick a form, fill the manifest-driven dynamic form, preview the render, create the repo on GitHub (or local-only).
- **Cloud** — GitHub repos across account and orgs: engine-readable template repos flagged (one code-search call), local clones cross-linked ("cloned" chip ↔ Clone/Folder actions), per-repo state preview: languages, branches, latest CI runs, README + docs rendered as sanitized markdown with in-reader link navigation.
- **Local** — recursive scan of the repositories folder (Templetry projects + plain git repos), organized by on-disk folders; per-repo preview: branches, remotes, last commit, working-tree state, docs.

Cross-cutting: GitHub sign-in via OAuth device flow (scopes `repo workflow`); full template update cycle — drift detection → assisted update → three-way merge; multi-catalog; in-app updater (v0.3.1: installer launches via ShellExecute so the UAC elevation prompt appears); contextual callouts wherever a section needs an action first (sign in, choose the repositories folder). Ships as Windows portable exe + NSIS installer (WebView2 runtime).

### Catalog & parents

- Model: **parents → forms → combinable features** ([ADR-0011](adr/0011-template-forms.md)), plus **lazy pieces** adopted post-creation ([ADR-0014](adr/0014-lazy-pieces.md)). Every form is a real project verified by CI ("Verify forms" workflows render + compile the combos on every push); `status: ready` in the registry is gated on green CI.
- **Eleven ecosystems** proven against one engine and one manifest schema: Kotlin Multiplatform, Android, React, Vue, Next.js, Go, Node/TypeScript, Python, Rust, JVM/Spring, .NET, plus the meta-template — the strongest evidence for the agnostic-engine thesis (ADR-0002). No engine change was needed for any of them.
- `meta/template` scaffolds new Templetry templates: pre-filled manifest, author guide (`GUIDE`), mechanism showcase, verify CI.

## Decisions in force

All ADRs are indexed in [`adr/README.md`](adr/README.md). Highlights: own engine in Go (0001, 0006), agnostic engine (0002), templates compile (0003), verify in containers (0004), updates prepared from day one (0005), multi-forge via BYOR (0009), parents/forms/features catalog (0011), desktop app with Wails superseding the hosted web app (0012 ⊃ 0010).

Multi-forge (ADR-0015, 2026-08-13): templates can be **hosted** on GitHub, GitLab or Gitea/Forgejo via source schemes; projects can be **created** on GitHub (OAuth), GitLab and Gitea/Forgejo (signed-in accounts, token in the OS keyring), or on literally any git host through BYOR (paste an empty repo's URL). The Cloud section lists repositories across every signed-in account.

Discarded along the way: the hosted web app.

## Specs

Normative documents in [`spec/`](spec/): [`template-yml.md`](spec/template-yml.md) (the manifest), [`directives.md`](spec/directives.md) (`tpl:` comment grammar, ADR-0007), [`answers-file.md`](spec/answers-file.md), [`validation-manifests.md`](spec/validation-manifests.md), [`compatibility.md`](spec/compatibility.md) (the v1 promise — normative from v1.0.0).

## Open backlog

- ~~Road to v1.0.0~~ **v1.0.0 declared and released 2026-08-07** across all components ([ADR-0013](adr/0013-declare-v1.md), soak window waived); the [compatibility policy](spec/compatibility.md) is in force.
- **1.x roadmap** planned in [study/roadmap-1x.md](study/roadmap-1x.md): update-in-CLI → AI/MCP surface → engine 1.1 additives → **lazy pieces** (study VII pending) → new parent → distribution.
- **Industrial pieces** researched in [study/industrial-pieces-v1.md](study/industrial-pieces-v1.md): identity/roles (NIST RBAC, SCIM 2.0), compliance (Verifactu — already mandatory in Spain), cross-cutting infrastructure, domain CRUDs and integrations, with a recommended build order.
