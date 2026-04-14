# metals-lsp

Scala language server ([Metals](https://scalameta.org/metals/)) for Claude
Code. Supports `.scala`, `.sbt`, `.sc`, and `.mill` files.

## Prerequisites

1. **JDK 17+** on your `PATH`.
2. **metals** installed via [Coursier](https://get-coursier.io/):
   ```bash
   cs install metals
   ```
3. **The workspace must be bootstrapped before first use.** See below.

## Important: bootstrap the workspace in VSCode first

Claude Code's LSP client does not implement `window/showMessageRequest`,
`workspace/configuration`, or `workspace/didChangeConfiguration`. Metals
uses these for first-time prompts like:

- "New sbt workspace detected, would you like to import the build?"
- "Multiple build definitions found. Which would you like to use?"

On a **cold workspace** (no `.bloop/`, no `.metals/metals.mv.db`) these
prompts fire and metals' init hangs — you get mtags-only fallback (no
cross-file navigation, spurious "not found: …" errors in all Scala files).

**Workaround — bootstrap the project in a full LSP client first:**

1. Install the
   [Scala (Metals) VSCode extension](https://marketplace.visualstudio.com/items?itemName=scalameta.metals).
2. Open your Scala project in VSCode.
3. Answer the prompts: pick your build tool (e.g. `sbt`), click
   "Import build", wait for `sbt bloopInstall` to finish (status bar
   shows progress).
4. Close VSCode.

The workspace now has `.bloop/*.json` + `.metals/metals.mv.db` with
`chosen_build_tool` recorded. Future Claude Code sessions on the same
workspace find this existing state and skip all prompts — the LSP just
works.

Any other full-featured Metals client (Emacs, nvim-metals, etc.) works
for this bootstrap step.

## Multi-project sbt builds with `ProjectRef`

If your root `build.sbt` references external sub-builds via
`ProjectRef(file("ext/Foo"), …)`, each referenced sub-directory needs
its own `project/metals.sbt` with the sbt-bloop plugin so that a single
root-level `bloopInstall` generates `.bloop/*.json` for **all**
sub-projects under the root `.bloop/`:

```scala
// project/metals.sbt (copy to each ext/<sub>/project/metals.sbt)
addSbtPlugin("ch.epfl.scala" % "sbt-bloop" % "2.0.19")
```

After adding these files, re-trigger "Import build" in VSCode. Verify
that the root `.bloop/` contains JSON files for every sub-project
(`ext/Foo`'s projects should land in `<root>/.bloop/`, not in
`<root>/ext/Foo/.bloop/`).

Once this is done, cross-sub-project navigation (goto-def, find-refs)
works in Claude Code as expected.

## Known limitations

- No headless cold-start support. Must bootstrap via a full LSP client
  (VSCode, Emacs, etc.) first. See the workaround above.
- `UserConfiguration` metals settings (`targetBuildTool`,
  `autoImportBuilds`, `sbtScript`, `defaultBspToBuildTool`, …) cannot
  be set from this plugin — Claude Code's LSP client does not forward
  them via `workspace/didChangeConfiguration`.
- No `src.zip` on your JDK ⇒ hover on `java.lang.*` types won't show
  documentation. Install with `sudo apt install openjdk-17-source`
  (or equivalent for your distribution).

## Links

- [Metals homepage](https://scalameta.org/metals/)
- [Metals on Coursier](https://get-coursier.io/docs/cli-install)
- [Metals GitHub](https://github.com/scalameta/metals)
