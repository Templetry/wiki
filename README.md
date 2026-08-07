# Templetry — Study & design wiki

**Templetry** is a manager for architectures, projects and frameworks: a tool that generates ready-to-work repositories from a library of multi-platform templates, and creates them on any git host (GitHub, GitLab, Gitea/Forgejo, self-hosted) at the speed of light.

> *Templetry* comes from **templet**, the original spelling of "template" — which also named a real weaving tool: the piece that keeps the cloth's shape on the loom.

## Project status

**Shipped (August 2026).** Engine v0.2.2 (hardened, remote catalogs, drift anchor) · `templetry` CLI with releases · verified template catalog (parents/forms/features, meta-template) · **Templetry Desktop** (Wails): GitHub sign-in, repo-first creation, preview, repo & project management, multi-catalog, full template update cycle (drift detection → assisted update → three-way merge) and in-app updater.

Discarded along the way: the hosted web app (ADR-0010 → desktop, ADR-0012) and the Gitea/GitLab adapters (study V archived — BYOR covers exotic hosts; adapters return if real users need them).

The full, always-current picture — org repos, versions, shipped capabilities, backlog — lives in [**state-of-the-art.md**](state-of-the-art.md).

## The three principles (study summary)

1. **The engine doesn't know what a framework is** — all knowledge lives in each template's `template.yml`; the engine has only 5 generic operations.
2. **Templates compile** — every template repo is a real project with its own CI, never a skeleton full of broken placeholders.
3. **Engine first, web later** — pure library + CLI; the web app (GitHub OAuth, catalog, forms) is a skin on top, in later phases.

## Repo map

| Folder | Contents |
|---|---|
| [`study/`](study/) | The full preliminary study (requirements, state of the art, risks, roadmap) |
| [`adr/`](adr/) | Architecture Decision Records — one decision per file, explicit status |
| [`spec/`](spec/) | The evolving spec of the `template.yml` manifest |
| [`journal/`](journal/) | Log of study and design sessions |

## How we work here

Decisions are recorded as ADRs with status `open → proposed → accepted` (or `superseded`). Nothing is decided "in someone's head": if it's not in an ADR, it's not decided. The [ADR index](adr/README.md) shows the global state at a glance.

## Roadmap

- ✅ Study · engine + CLI · template CI verification · catalog (parents/forms/features) · meta-template & third-party catalogs · desktop app · template updates incl. three-way merge · app auto-update.
- Open backlog: planned form `android/single-module`, engine `verify` in Docker, settings sync via GitHub, code signing.
- Next milestone: **engine v1.0.0** — gates and timeline in [study/road-to-v1.md](study/road-to-v1.md).
