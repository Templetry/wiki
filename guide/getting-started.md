# Getting started

## Install

**macOS and Linux (Homebrew)** — installs the CLI and the MCP server:

```sh
brew install Templetry/tap/templetry
```

**macOS and Linux (script)** — the same two binaries, verified against the release's own `SHA256SUMS`, no package manager:

```sh
curl -fsSL https://raw.githubusercontent.com/Templetry/engine/main/install.sh | sh
```

It installs into `/usr/local/bin` when that is writable and `~/.local/bin` otherwise, and tells you if that directory is not on your `PATH`. Set `TEMPLETRY_BIN_DIR` to choose, or `TEMPLETRY_VERSION` to pin a tag.

**Windows (scoop)** — installs the CLI and the MCP server:

```powershell
scoop bucket add templetry https://github.com/Templetry/scoop-bucket
scoop install templetry
```

**Any platform (binary)** — download your build from [engine releases](https://github.com/Templetry/engine/releases/latest), rename it to `templetry` (or `templetry.exe`) and put it on your `PATH`. Checksums are published as `SHA256SUMS`.

**With the Go toolchain**:

```sh
go install github.com/Templetry/engine/cmd/templetry@latest
```

**Prefer a UI?** Install [Templetry Desktop](https://github.com/Templetry/desktop/releases/latest) (Windows, Linux, macOS) and read [the desktop guide](desktop.md) instead. Nothing below is required for it.

Check it works:

```sh
templetry version
```

## Your first project in 60 seconds

See what the catalog offers:

```sh
templetry list
```

Generate one. `init` fetches the form from its repository and renders it locally:

```sh
templetry init python/fastapi-users --out ./my-api \
  --set "project_name=My Api"
```

That is a complete, working project — install and run its tests right away:

```sh
cd my-api
pip install -e .[dev]
pytest
```

## What you just got

```
my-api/
  src/my_api/…                 renamed to your project, not "template_app"
  tests/…
  .templetry-answers.yml       what made this project, and from which commit
```

That last file is the interesting one:

```yaml
schema_version: 1
template:
  name: python-fastapi-users
  source: "github.com/Templetry/python@main/fastapi-users"
  commit: 8f21c0…            # the exact template commit you rendered
variables:
  project_name: My Api
```

Keep it in git. It costs nothing and it is what makes [updates](keeping-updated.md) and [pieces](using-pieces.md) possible later. Delete it and the project becomes an ordinary folder — still perfectly usable, just disconnected.

## Next steps

- Choose features and presets deliberately → [Using templates](using-templates.md)
- Add RBAC, an audit trail or dependency automation → [Using pieces](using-pieces.md)
- Create the repository on GitHub, GitLab or anywhere → [Multi-forge](multi-forge.md)
