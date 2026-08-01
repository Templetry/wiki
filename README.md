# Templetry — Study & design wiki

**Templetry** is a manager for architectures, projects and frameworks: a tool that generates ready-to-work repositories from a library of multi-platform templates, and creates them on any git host (GitHub, GitLab, Gitea/Forgejo, self-hosted) at the speed of light.

> *Templetry* comes from **templet**, the original spelling of "template" — which also named a real weaving tool: the piece that keeps the cloth's shape on the loom.

## Project status

**Phase 0 — Preliminary study.** No code yet, by design: the decisions that are expensive to reverse get closed first.

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

- **Phase 0 — Study** *(current)*: close the open decisions, dry-run-validate the manifest against 3 dissimilar templates.
- **Phase 1 — Engine**: library + CLI (`templetry render`, `templetry plan`), golden tests.
- **Phase 2 — Verify + CI**: container-based verification, first real templates migrated.
- **Phase 3 — Web MVP**: GitHub OAuth, catalog, dynamic form, repo creation (GitHub + bring-your-own-remote).
- **Phase 4+**: GitLab/Gitea adapters, three-way-merge updates, third-party templates.
