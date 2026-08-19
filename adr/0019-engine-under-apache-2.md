# ADR 0019 — The engine moves to Apache License 2.0

**Status:** ✅ Accepted · 2026-08-19

## Context

The engine shipped under **PolyForm Noncommercial 1.0.0**, which permits any noncommercial purpose and requires the author's permission for anything else. Every other repository in the organization is MIT.

A competitive study of the tools people actually use put the cost of that choice in plain terms:

| Project | Licence |
|---|---|
| backstage/backstage | Apache-2.0 |
| cookiecutter/cookiecutter | BSD-3-Clause |
| Rich-Harris/degit | MIT |
| yeoman/yo | BSD-2-Clause |
| copier-org/copier | MIT |
| projen/projen | Apache-2.0 |

Six comparable tools, six permissive licences, no exceptions. Against that field a noncommercial engine barred exactly the people the project needs:

- **Someone using it at work** — the single most common context for scaffolding a new service.
- **Any company evaluating it**, at the first question their legal review asks.
- **Any contributor**, who would spend evenings on a tool they could not then use in the day job. That is not a bargain anyone accepts twice.

The templates and the desktop app were already MIT, so the restriction sat on the one component every path runs through — and it was invisible until someone had already invested time.

## Decision

**The engine is licensed under Apache License 2.0.** The rest of the organization stays MIT.

Apache rather than MIT for two reasons that matter to this particular project:

- **An explicit patent grant.** The engine performs non-obvious operations — identity-map renaming with derived casings, three-way template merges. Apache grants contributors' patent rights to users and terminates them for anyone who sues over patents; MIT is silent on the subject.
- **A trademark reservation.** Apache §6 states plainly that the licence does not grant rights to use the licensor's names or marks. *Templetry* is a brand with [written guidelines](../brand/guidelines.md); the code being free to fork does not make the name free to reuse, and Apache says so without a separate document.

## Timing

Relicensing needs the agreement of every copyright holder. Today the engine has exactly one — a single author across all 34 commits, zero forks, zero external contributors.

**This was the cheapest this decision would ever be.** With one outside contributor it needs their permission; with twenty it becomes a project of its own. Deciding it before asking anyone to contribute is not a detail of sequencing — it is the only order that works.

## Consequences

- Templetry can be used commercially, which is what the majority of its potential users need.
- The option of charging for the engine itself is gone. Support, hosted catalogs and the brand remain available if that question ever returns.
- `LICENSE.md` becomes `LICENSE` (the Apache convention), with a `NOTICE` file alongside it.
- Every surface that stated the old licence was corrected in the same change: the engine README, the scoop manifest's SPDX field, the state of the art, and the v1.x roadmap.

This ADR records the change deliberately rather than letting a quiet file swap stand as the only trace, because a licence is a promise to other people and the reasoning behind it is part of the promise.
