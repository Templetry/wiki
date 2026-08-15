# Templetry — Study & design wiki

**Templetry** is a manager for architectures, projects and frameworks: it generates ready-to-work repositories from a library of multi-platform templates, creates them on any git host (GitHub, GitLab, Gitea/Forgejo, self-hosted), and keeps them alive afterwards — projects pull template improvements and adopt new pieces long after creation.

> *Templetry* comes from **templet**, the original spelling of "template" — which also named a real weaving tool: the piece that keeps the cloth's shape on the loom.

## Project status

**v1.0.0 and beyond (August 2026).** Engine **v1.10.0** (library + `templetry` CLI + `templetry-mcp` server), desktop **v1.7.1** (Windows, Linux, macOS), a catalog of **22 forms across 10 parents**, **12 form-local pieces and 2 common pieces** covering eleven ecosystems, all CI-verified.

New here? Start with the [**usage guides**](guide/). The full, always-current picture — repos, versions, capabilities, backlog — lives in [**state-of-the-art.md**](state-of-the-art.md).

## The principles

1. **The engine doesn't know what a framework is** — all knowledge lives in each template's `template.yml`; the engine has only generic operations ([ADR-0002](adr/0002-knowledge-lives-in-the-manifest.md)). Eleven ecosystems later, it has never needed a framework-specific line.
2. **Templates compile** — every template is a real project whose CI renders *and builds* its output ([ADR-0003](adr/0003-templates-compile.md)). Defects surface in the template's CI, not in a user's project.
3. **Generated projects stay alive** — the answers file records template, commit and inputs from day one ([ADR-0005](adr/0005-prepare-for-updates.md)), which is what makes three-way-merge updates and lazy pieces possible.
4. **Encode standards, not opinions** — an industrial piece is worth shipping when it encodes a real specification (NIST RBAC, SCIM, RD 1007/2023); otherwise it becomes someone else's technical debt ([study VIII](study/industrial-pieces-v1.md)).

## The model

**Parent → form → feature → piece** ([ADR-0011](adr/0011-template-forms.md), [ADR-0014](adr/0014-lazy-pieces.md)):

- **Parent**: one repo per concept (`kmp`, `web`, `python`…).
- **Form**: a structural variant, chosen not combined — a subdirectory that compiles on its own with its own `template.yml`.
- **Feature**: the freely combinable axis inside a form, resolved at render time (with `requires`/`conflicts` and named `presets`).
- **Piece**: a decoupled unit adopted *after* creation, with its own variables, identity map and drift anchor.

Golden rule: **additive → feature · structural → form · independent lifecycle → piece.**

## Repo map

| Folder | Contents |
|---|---|
| [`guide/`](guide/) | Usage guides: getting started, templates, pieces, updates, desktop, authoring, agents, multi-forge |
| [`study/`](study/) | Research: engine design, technology choice, execution model, template forms, multi-forge, road to v1, roadmap 1.x, lazy pieces, industrial pieces, Verifactu |
| [`adr/`](adr/) | Architecture Decision Records — one decision per file, explicit status |
| [`spec/`](spec/) | Normative specs: manifest, directives, answers file, `piece.yml`, compatibility policy |
| [`journal/`](journal/) | Log of working sessions and what they taught |
| [`brand/`](brand/) | Ink & Brass: palette, typography, logo, voice |

## How we work here

Decisions are recorded as ADRs with status `open → proposed → accepted` (or `superseded`). Nothing is decided "in someone's head": if it's not in an ADR, it's not decided. Research that precedes a decision goes to `study/`. The [ADR index](adr/README.md) shows the global state at a glance.

Since v1.0.0 the [compatibility policy](spec/compatibility.md) is in force: the manifest, directives, answers file, registry v2, CLI surface and exported Go API only break with a major version.
