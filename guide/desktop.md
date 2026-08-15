# The desktop app

[Templetry Desktop](https://github.com/Templetry/desktop/releases/latest) does everything the CLI does, plus browsing your repositories. Windows (installer or portable), Linux and macOS (universal). It embeds the engine — there is no separate CLI to install and no server involved.

> On Windows, SmartScreen will warn about the installer: the builds are not code-signed yet. Choose *More info → Run anyway*, or use the portable exe.

## Build

Create a project:

1. Pick a form from the catalog sidebar.
2. Fill the form — it is generated from the template's manifest, so it always matches the template: variables with their patterns, feature checkboxes, and a **preset** row when the template offers combinations.
3. **Preview** renders the project in memory and lets you read any file before writing anything.
4. Choose the destination and create.

The destination picker offers three kinds:

- **A namespace from a signed-in account** — the app creates the repository for you and pushes.
- **Any git host** — paste the URL of an empty repository you created yourself; the app renders, commits and pushes. This works with every git server that exists.
- **Local only** — no remote at all.

## Cloud

Your repositories across every signed-in account, with:

- a **template** chip on repos the engine can render (they carry a `template.yml`),
- a **cloned** chip on repos already on your disk, with *Folder* instead of *Clone*,
- a **preview** per repo: description, languages, branches, the latest CI runs with their status, and a reader for the README and any markdown docs.

## Local

Every repository under your repositories folder, found recursively and grouped by the folders they live in. Per project:

- branch, remotes, last commit and whether the working tree is clean,
- a **template updated** chip when the template moved past what the project recorded,
- **Preview update** → the same diff/merge cycle as the CLI, file by file, before writing,
- a **Pieces** panel: what the template offers, what is already applied, the variables each piece takes, and one click to adopt.

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
