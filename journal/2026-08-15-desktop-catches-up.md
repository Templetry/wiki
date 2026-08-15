# 2026-08-15 — The desktop catches up, and the shape of a familiar bug

Two unrelated pieces of work, and a pattern worth naming that connects them to ADR-0015.

## Guides, and a documentation pass

The wiki gained a [`guide/`](../guide/) set: getting started, using templates, using pieces, keeping projects updated, the desktop, authoring templates, authoring pieces, AI agents and MCP, multi-forge. Until now everything explained *why* Templetry is the way it is — ADRs, studies, specs — and nothing explained how to use it. Both audiences are real and they are not the same person.

The same pass found the usual rot: version numbers stale in three places, [study X](../study/where-next-v1.md) still describing shipped work as a proposal, and the engine README claiming five MCP tools when there are seven. Versions now live in exactly two files, so there is one place to update.

## `dotnet/razor-web`

A Razor Pages form next to `minimal-api`. Its interesting part is not the framework but the **nav socket**: the layout builds its navigation by discovering `INavContributor` implementations at startup, so the optional contact page joins the menu by existing. Removing the feature removes its page, its link and its tests without touching a shared file.

That was forced by a limitation: `.cshtml` is not in the directive scanner's comment table, so `tpl:if` could not reach the layout. The workaround turned out better than the thing it replaced — the form is now piece-ready, and a piece adding a page needs no wiring at all. Worth remembering when the scanner's table looks like the obvious fix.

## The pattern: shipped code implying a capability

An audit of the desktop against engine v1.9.0 turned up six divergences. Individually they look like backlog. Together they are one shape, and it is the second time:

> ADR-0015 was written because BYOR had been *accepted* and never implemented — the v1 claim "BYOR covers every host" described a design, not shipped code.

This time the app carried multi-forge accounts, listed repositories from all of them, and then called `api.github.com` for everything you could do with a row. Opening a GitLab repo's preview returned a 404. Pushing to a non-GitHub repo blanked its credentials and fell back to the user's git helper, which `GIT_TERMINAL_PROMPT=0` turns into a silent failure. The capability was half-present, which is worse than absent: absent is honest.

Same shape for pieces. `piece.Available` and `piece.Resolve` shipped in engine v1.9.0; the desktop stayed on v1.8.0 calling `piece.List`, so the shared catalog's pieces were adoptable from the CLI and invisible in the app. And drift checked the template's anchor while ignoring the per-piece anchors the engine had been recording since v1.8.0.

**The tell in all three: a consumer that lags the library it embeds.** The CLI and the MCP server both used `source.ParseRef` + `source.Fetch`; only the desktop still used the GitHub-specific call. When two of three consumers agree and one does not, the odd one is not a design choice.

Fixed in [desktop v1.7.0](https://github.com/Templetry/desktop/releases/tag/v1.7.0), which also closed the two remaining gaps: `verify` had no binding at all (the only engine verb with none), and feature `requires`/`conflicts` were invisible in the UI, so an invalid combination looked fine until the engine refused it.

## Two things the fixes taught

**Bump the engine first, then look.** Moving the desktop from v1.8.0 to v1.9.0 compiled with zero changes. That is the third data point for the v1 compatibility promise, and it means "which engine is the app on" is a cheap question to ask on a schedule rather than an expensive one to discover.

**Recording the wrong source would have broken updates silently.** A common piece has to record *its own* repository in the answers file. The obvious code — reuse the form's source, as the form-local path does — produces a valid-looking record that makes `templetry update` look for the fix where it was never made. It would have failed at the worst moment: months later, quietly.

## The same shape, one level up: the pinned CLI

Study X's phase 2 asked for "one pinned CLI version, centrally managed". It shipped as *bumping ten repositories by hand* — and by the time the desktop audit ran, all seventeen workflow references were on v1.7.0 against an engine at v1.9.0, `meta` had quietly been tracking `latest` instead of a pin, and `kmp` carried its own copy of the OS detection.

That is not ten stale files. It is a mechanism that was never built, doing what an unbuilt mechanism does.

The fix is a composite action in the org's `.github` repo: parents call `setup-templetry@main`, the version lives in its `action.yml`, and one edit moves the catalog. Renovate follows that single line against the engine's releases, so the bump arrives as a PR the parents' CI judges — the same loop already used for template dependencies, pointed at the tool instead of the libraries.

**Moving all ten from v1.7.0 to v1.9.0 in one push: ten green.** Eleven ecosystems, two engine minors, no template touched. Fourth data point for the v1 promise, and the first one that was free — the mechanism did it, not a person.

## Takeaway

When an ADR or a study accepts something, acceptance is not the end of the work. Twice today the gap was the same: a decision recorded, an implementation that approximated it, and drift that arrived on schedule.

The cheap tell is comparing *consumers*. Two of three used `source.ParseRef`; one did not. Nine of ten pinned a version; one tracked `latest`. When siblings disagree, the odd one is rarely a design choice — and a `grep` finds it in a minute.

And when the manual version of a mechanism drifts within a day of being finished, that is the signal to build the mechanism, not to redo the work.
