# The desktop app

[Templetry Desktop](https://github.com/Templetry/desktop/releases/latest) does everything the CLI does, plus browsing your repositories. Windows (installer or portable), Linux and macOS (universal). It embeds the engine — there is no separate CLI to install and no server involved.

> On Windows, SmartScreen will warn about the installer: the builds are not code-signed yet. Choose *More info → Run anyway*, or use the portable exe.

## Build

Create a project:

1. Pick a form from the catalog sidebar. Every form shows its **taxonomy** — what it is (`kinds`, in brass), what it is written in, what it is built on — and the sidebar filters on it: click kind chips to narrow, or type to search across names, descriptions and all three axes.
2. Fill the form — it is generated from the template's manifest, so it always matches the template: variables with their patterns, feature checkboxes, and a **preset** row when the template offers combinations. Features that declare `requires` or `conflicts` say so, and the actions stay blocked until the combination is valid — the engine refuses to guess which side you meant, and neither does the app.
3. **Preview** renders the project in memory and lets you read any file before writing anything.
4. **Verify build** goes one step further: it renders your inputs to a temporary directory and builds them inside the manifest's container, streaming the log. Needs Docker; nothing is installed on your machine and nothing is written to your folders.
5. Choose the destination and create.

The destination picker offers three kinds:

- **A namespace from a signed-in account** — the app creates the repository for you and pushes.
- **Any git host** — paste the URL of an empty repository you created yourself; the app renders, commits and pushes. This works with every git server that exists.
- **Local only** — no remote at all.

## Cloud

Your repositories across every signed-in account — GitHub, GitLab and Gitea/Forgejo alike — with:

- a **template** chip on repos the engine can render (they carry a `template.yml`), and inside the preview each form it found, with the name, description and taxonomy read from its manifest,
- a **cloned** chip on repos already on your disk, with *Folder* instead of *Clone*,
- a **preview** per repo: description, languages, branches, the latest CI runs or pipelines with their status, and a reader for the README and any markdown docs.

Cloning authenticates as the account the repository came from, so private repos on any signed-in forge clone too.

## Local

Every repository under your repositories folder, found recursively and grouped by the folders they live in. Per project:

- branch, remotes, last commit and whether the working tree is clean,
- a chip when there is something to pull — the template moved past what the project recorded, or an applied **piece** did (each piece tracks its own source, so a fix released in a common piece shows up here),
- **Preview update** → the same diff/merge cycle as the CLI, file by file, before writing,
- a **Pieces** panel: everything the project can adopt — the pieces its own form ships plus the **common** ones from your catalogs that fit its stack — with the variables each takes and one click to adopt.

## Accounts

Settings → Accounts:

- **GitHub** signs in from the top bar with the OAuth device flow — no password, no token to manage.
- **GitLab** and **Gitea/Forgejo** take a personal access token per host (`gitlab.com`, `codeberg.org`, your company server). Tokens are validated against the API and stored in the **OS keyring**, never in the settings file — exports stay shareable.

Why a token rather than OAuth for those: their device flows require registering an application on each instance, which cannot be shipped centrally for self-hosted servers. A token works everywhere from the first minute.

## Settings worth knowing

- **Repositories folder** — where clones and new projects go, and what the Local section scans.
- **Catalogs** — add any `registry.json`; your catalog behaves exactly like the official one.
- **Appearance** — theme, accent, density, interface scale, preview layout.
- **Export / Import** — settings travel as JSON without tokens.

## Updates

Settings → About checks published stable releases and can install a new version in place (Windows; elsewhere download from releases). The embedded engine is not separately updatable — it ships inside the app, so a newer engine arrives with a newer app.
