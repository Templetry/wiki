# Templetry — State of the art

**Snapshot: 2026-08-07** · maintained as the single up-to-date picture of what exists, where it lives and what state it is in. History lives in [`journal/`](journal/); decisions in [`adr/`](adr/).

## What Templetry is

Templetry generates ready-to-work repositories from a library of **compilable** multi-platform templates and creates them on any git host. The engine is a pure Go library + CLI; the desktop app embeds that library behind a native UI. All framework knowledge lives in each template's `template.yml` manifest — the engine has only generic operations ([ADR-0002](adr/0002-knowledge-lives-in-the-manifest.md), [ADR-0003](adr/0003-templates-compile.md)).

## Where it lives

Everything is published in the GitHub organization **[Templetry](https://github.com/Templetry)** (migrated from personal repos in August 2026; the pre-org repos `android-native-base`, `kmp-native-base` and `compose-multiplatform-app` were folded into the parents and deleted).

| Repo | Role | State |
|---|---|---|
| [engine](https://github.com/Templetry/engine) | Pure Go library + `templetry` CLI (`list`, `init`, `plan`, `render`, `version`) | **v0.2.2** — binary releases for linux/darwin/windows (amd64 + arm64) |
| [desktop](https://github.com/Templetry/desktop) | Native desktop app (Wails: Go backend embedding the engine, React/TS frontend) | **v0.2.0** released; `main` carries unreleased v0.3 work (three sections, local scan, previews). The v0.2.1 tag was dropped during the 2026-08-06 GitHub Actions outage — next release retags from `main` |
| [catalog](https://github.com/Templetry/catalog) | Default template registry — `registry.json` (schema v2) consumed by CLI and desktop | 6 forms `ready`, 1 `planned` |
| [kmp](https://github.com/Templetry/kmp) | Parent: Kotlin Multiplatform | forms `modular-features`, `single-module`, `modular-ui` — all ready, CI-verified |
| [android](https://github.com/Templetry/android) | Parent: Android native | form `modular-features` ready; `single-module` planned |
| [meta](https://github.com/Templetry/meta) | Parent: the template that creates Templetry templates | form `template` ready |
| [wiki](https://github.com/Templetry/wiki) | This repo: studies, ADRs, specs, journal, brand | living |
| [.github](https://github.com/Templetry/.github) | Organization profile | living |

Local layout mirrors the org under `Repos\templetry\` with the parents grouped in `parents\`.

## Shipped capabilities

### Engine + CLI (v0.2.2)

- Manifest parsing + validation + derived casings; planner (feature exclusion, identity renaming); renderer (directive scanner, JSON patches); deterministic output — same inputs, byte-identical result.
- Remote catalogs: `templetry list` / `templetry init parent/form` resolve against the official registry (`https://raw.githubusercontent.com/Templetry/catalog/main/registry.json`) or any custom one.
- Answers file records every input **and the resolved template commit** (drift anchor) — the foundation for the update cycle ([ADR-0005](adr/0005-prepare-for-updates.md)).
- Releases with SHA256SUMS; CI on every push.

### Desktop (v0.2.0 released · v0.3 on main)

Three main sections:

- **Build** — repo-first creation: pick a form, fill the manifest-driven dynamic form, preview the render, create the repo on GitHub (or local-only).
- **Cloud** — GitHub repos across account and orgs: engine-readable template repos flagged (one code-search call), local clones cross-linked ("cloned" chip ↔ Clone/Folder actions), per-repo state preview: languages, branches, latest CI runs, README + docs rendered as sanitized markdown with in-reader link navigation.
- **Local** — recursive scan of the repositories folder (Templetry projects + plain git repos), organized by on-disk folders; per-repo preview: branches, remotes, last commit, working-tree state, docs.

Cross-cutting: GitHub sign-in via OAuth device flow (scopes `repo workflow`); full template update cycle — drift detection → assisted update → three-way merge; multi-catalog; in-app updater; contextual callouts wherever a section needs an action first (sign in, choose the repositories folder). Ships as Windows portable exe + NSIS installer (WebView2 runtime).

### Catalog & parents

- Model: **parents → forms → combinable features** ([ADR-0011](adr/0011-template-forms.md)). Every form is a real project verified by CI ("Verify forms" workflows render + compile the combos on every push); `status: ready` in the registry is gated on green CI.
- `meta/template` scaffolds new Templetry templates: pre-filled manifest, author guide (`GUIDE`), mechanism showcase, verify CI.

## Decisions in force

All ADRs are indexed in [`adr/README.md`](adr/README.md). Highlights: own engine in Go (0001, 0006), agnostic engine (0002), templates compile (0003), verify in containers (0004), updates prepared from day one (0005), multi-forge via BYOR (0009), parents/forms/features catalog (0011), desktop app with Wails superseding the hosted web app (0012 ⊃ 0010).

Discarded along the way: the hosted web app and dedicated Gitea/GitLab adapters (BYOR covers exotic hosts; adapters return if real users need them).

## Specs

Normative documents in [`spec/`](spec/): [`template-yml.md`](spec/template-yml.md) (the manifest), [`directives.md`](spec/directives.md) (`tpl:` comment grammar, ADR-0007), [`answers-file.md`](spec/answers-file.md), [`validation-manifests.md`](spec/validation-manifests.md).

## Open backlog

- Form `android/single-module` (registry `planned`).
- ~~Engine `verify` in Docker~~ done 2026-08-07: `templetry verify` renders (or takes `--dir`) and runs the manifest's verify in a container — ADR-0004 fully realized.
- ~~Engine hardening~~ done 2026-08-07: dest paths can't escape or abuse platform semantics (incl. Windows device names and case-insensitive collisions); directive scanner fuzz-tested; manifests tolerate a UTF-8 BOM. Pending release as engine v0.3.0.
- Desktop: tag the v0.3 release; settings sync via GitHub; code signing; macOS/Linux builds.
- Road to v1.0.0: gates and proposed timeline in [study/road-to-v1.md](study/road-to-v1.md).
