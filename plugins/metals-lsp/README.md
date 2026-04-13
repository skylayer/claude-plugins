# metals-lsp

Scala language server ([Metals](https://scalameta.org/metals/)) for Claude Code, providing code intelligence across Scala 2.x / Scala 3 sources, `sbt` build files, worksheets, and `mill` build files.

## Supported Extensions
`.scala`, `.sbt`, `.sc`, `.mill`

## Installation

Metals is distributed via [Coursier](https://get-coursier.io/).

```bash
# Install coursier if you don't have it
curl -fL https://github.com/coursier/coursier/releases/latest/download/cs-x86_64-pc-linux.gz | gzip -d > cs && chmod +x cs && ./cs setup

# Install metals
cs install metals
```

On macOS:

```bash
brew install coursier/formulas/coursier
cs install metals
```

Make sure `metals` is on your `PATH` — `cs install` writes to `~/.local/share/coursier/bin` by default.

## Requirements

- **JDK 17+** on `PATH` (Metals uses it both to run itself and to resolve build tooling). `JAVA_HOME` is respected if set.
- A supported build tool: `sbt`, `mill`, `gradle`, `maven`, or a Bloop export. Metals imports the build on first open, which is why `startupTimeout` is set to 5 minutes — the very first import in a large project may download the full dependency graph.

## Notes

- For `sbt` projects Metals invokes `sbt bloopInstall` in the background; expect a one-time delay the first time you open a new project.
- `.sc` files are treated as Scala worksheets or Ammonite scripts depending on context.
- `.mill` support requires a recent Metals version (≥ 1.3).

## More Information

- [Metals homepage](https://scalameta.org/metals/)
- [GitHub repository](https://github.com/scalameta/metals)
- [Metals on Coursier](https://get-coursier.io/docs/cli-install)
