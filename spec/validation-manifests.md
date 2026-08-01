# Dry-run validation — three dissimilar manifests

Study I §7.1: hand-write the manifest for three dissimilar templates; if any needs hacks, the schema is wrong. Verdict at the bottom. (The third template, React, lives as the engine's golden-test fixture `testdata/react-mini`.)

## 1. Kotlin Multiplatform

```yaml
schema_version: 1
name: kmp-compose-app
platform: multiplatform
framework: kmp
variables:
  - key: project_name
    label: Project name
    type: string
    pattern: "^[A-Za-z][A-Za-z0-9 ]*$"
  - key: base_package
    label: Base package
    type: string
    pattern: "^[a-z]+(\\.[a-z][a-z0-9]*)+$"
  - key: min_sdk
    type: select
    options: ["24", "26", "29"]
    default: "24"
identity:
  - from: "com.template.base"      # renames deep source dirs too
    to: "{base_package}"
  - from: "TemplateApp"
    to: "{project_name.pascal}"
  - from: "template-app"
    to: "{project_name.kebab}"
features:
  - key: room
    label: Room persistence
    default: false
    files: ["core/database/**"]
  - key: ktor_client
    label: Ktor networking
    default: true
    files: ["core/network/**"]
verify:
  image: gradle:8-jdk17
  run: ./gradlew build
```

Notes: `min_sdk` reaches `build.gradle.kts` via `tpl:var min_sdk 24` (line-scoped, no dangerous global "24" replace). Room's version-catalog entries go in with `tpl:if room` inside `libs.versions.toml` (`#` comments) — no patch needed. **Zero hacks.**

## 2. Python / FastAPI

```yaml
schema_version: 1
name: fastapi-service
platform: backend
framework: fastapi
variables:
  - key: project_name
    label: Project name
    type: string
  - key: python_version
    type: select
    options: ["3.12", "3.13"]
    default: "3.13"
identity:
  - from: "template_app"           # snake_case package dir under src/
    to: "{project_name.snake}"
  - from: "template-app"           # pyproject name, docker image tag
    to: "{project_name.kebab}"
  - from: "TemplateApp"            # docstrings, OpenAPI title
    to: "{project_name.pascal}"
features:
  - key: postgres
    label: PostgreSQL (SQLAlchemy + Alembic)
    default: false
    files: ["src/template_app/db/**", "alembic/**", "alembic.ini"]
  - key: docker
    label: Dockerfile + compose
    default: true
    files: ["Dockerfile", "compose.yaml"]
verify:
  image: python:3.13-slim
  run: pip install -e .[dev] && pytest
```

Notes: everything is `#`-commented (pyproject.toml included) → directives cover conditional deps; no JSON in the whole template, patches unused. The `files` globs of `postgres` mix directories and a single file — the glob rule handles both. **Zero hacks.**

## Verdict

Three ecosystems, one schema, no escape hatches used. The schema holds. Two observations feeding v1.1 (recorded, not blocking): feature `files` listing a snake_case path that the identity map renames (`src/template_app/db/**`) means globs are matched against **template paths, not renamed paths** — now normative in the spec; and TOML patches would occasionally be nicer than `tpl:if` in `pyproject.toml` (v1.1 candidate).
