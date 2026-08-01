# Architecture Decision Records

One decision per file. Format: context → decision → consequences, with an explicit status in the header.

Statuses: **open** (undecided) · **proposed** (candidate, pending validation) · **accepted** (in force) · **superseded by ADR-XXXX**.

## Index

| ADR | Title | Status |
|---|---|---|
| [0001](0001-own-engine.md) | Build our own engine instead of adopting Copier/Cookiecutter | ✅ Accepted |
| [0002](0002-knowledge-lives-in-the-manifest.md) | The engine is agnostic; knowledge lives in the manifest | ✅ Accepted |
| [0003](0003-templates-compile.md) | Templates compile; three transformation mechanisms | ✅ Accepted |
| [0004](0004-verify-in-containers.md) | Verification runs in containers, declared in the manifest | ✅ Accepted |
| [0005](0005-prepare-for-updates.md) | Answers file from day one; three-way-merge updates postponed | ✅ Accepted |
| [0006](0006-engine-language.md) | Engine implementation language: Go | ✅ Accepted |
| [0007](0007-directive-grammar.md) | Minimal grammar for `tpl:` directives | ✅ Accepted |
| [0008](0008-project-name-templetry.md) | Project name: Templetry | ✅ Accepted |
| [0009](0009-multi-forge.md) | Repo creation on any forge (multi-forge) | ✅ Accepted |
| [0010](0010-engine-execution-model.md) | Engine execution model: embedded in a Go backend | 🟡 Proposed |
