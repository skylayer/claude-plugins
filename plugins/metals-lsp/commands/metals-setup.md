---
name: metals-setup
description: Bootstrap metals-lsp for the current Scala workspace (runs sbt bloopInstall + propagates to submodules referenced via ProjectRef).
argument-hint: (none — operates on current workspace)
---

The user ran `/metals-setup`. metals-lsp can't cold-start on an
un-bootstrapped sbt project because Claude Code's LSP client doesn't
implement `window/showMessageRequest`, which metals uses to prompt
"New sbt workspace detected, would you like to import the build?".
Your job is to bootstrap the build ahead of time so metals finds an
existing `.bloop/` on startup and skips the prompt entirely.

Follow these steps in order.

## Step 1 — Bootstrap the root project

Run `metals-setup-project` from the workspace root (the directory
containing the root `build.sbt`):

```bash
cd <workspace-root>
metals-setup-project
```

This script:
- writes `project/metals.sbt` with the sbt-bloop plugin (if missing)
- runs `sbt bloopInstall` (slow first time: ~30s–3min; fast on re-runs)
- verifies `.bloop/*.json` files were generated

If it fails, read the error output and report it to the user before
continuing. Common causes: `sbt` not on PATH (`cs install sbt`), corrupt
build.sbt, network/proxy issues during Ivy resolution.

## Step 2 — Find external sub-projects

Read the root `build.sbt`. Look for references to **external** sub-projects
— i.e., directories that have their own independent `build.sbt`:

- `lazy val X = ProjectRef(file("path/to/subdir"), "name")`
- `lazy val X = RootProject(file("path/to/subdir"))`
- `dependsOn(ProjectRef(file("..."), ...))`

**Do NOT** include in-build modules like `lazy val X = project.in(file("..."))`
that are part of the same sbt build — those are already handled by the
root `bloopInstall`. You only need to recurse into directories that have
their own `build.sbt` and thus their own separate sbt build.

Check: for each candidate path, does `<path>/build.sbt` exist and is it a
separate build (not the same as the root)?

If there are no external sub-projects, skip to Step 4.

## Step 3 — Bootstrap each sub-project

For each external sub-project directory found in Step 2:

```bash
cd <sub-project-path>
metals-setup-project
```

Each sub-project gets its own `.bloop/*.json` generated in its own
directory. metals will find these when indexing.

## Step 4 — Verify and reindex

- Confirm `.bloop/` exists at the root with `*.json` files inside.
- Confirm each bootstrapped sub-project also has its own `.bloop/`.
- Trigger any LSP call (e.g., `hover` on a `.scala` file). metals should
  now start cleanly, connect to bloop, and begin indexing without any
  prompts.

Report to the user: how many projects were bootstrapped and where, and
confirm that LSP is now working.

## Notes

- This setup only needs to run **once per workspace**. metals persists its
  `.bloop/` connection state and future cold starts are instant.
- If the user later adds a new external sub-project, re-run
  `metals-setup-project` in that new directory only.
- `metals-setup-project` is idempotent — it's safe to re-run anywhere.
