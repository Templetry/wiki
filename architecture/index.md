# Organization architecture

A repo-level map of every [Templetry](https://github.com/Templetry) repository and how they depend on each other: the engine and desktop as the two things people run, the catalog and pieces as the content they read, and the governance loop (Renovate → `.github`'s pinned action → each parent's CI) that keeps all eleven parents and both distribution channels current.

Pan, zoom, search and trace relationships inside the diagram. Three guided views are pinned in the top bar: **Usage path**, **Content pipeline**, and **Governance & distribution**.

<iframe src="organization-architecture.html" title="Templetry organization architecture" style="width:100%; height:85vh; border:1px solid var(--md-default-fg-color--lightest, #2c303d); border-radius:4px;" loading="lazy"></iframe>

[Open the diagram in its own tab](organization-architecture.html){ target="_blank" }

See [state-of-the-art.md](../state-of-the-art.md) for the prose version of the same picture, and the [ADR index](../adr/README.md) for the decisions that produced it.
