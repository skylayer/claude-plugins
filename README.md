# claude-plugins

Personal [Claude Code](https://code.claude.com) plugin marketplace.

## Installation

Add this marketplace to your Claude Code:

```
/plugin marketplace add skylayer/claude-plugins
```

Then browse plugins:

```
/plugin
```

or install directly:

```
/plugin install <plugin-name>@skylayer-plugins
```

## Plugins

| Name | Description |
|---|---|
| [`metals-lsp`](./plugins/metals-lsp) | Scala language server (Metals) for `.scala`, `.sbt`, `.sc`, `.mill` |

## Structure

```
.
├── .claude-plugin/
│   └── marketplace.json    # marketplace manifest
├── plugins/
│   └── metals-lsp/         # individual plugin dirs
│       └── README.md
└── README.md
```

## License

MIT. See [LICENSE](./LICENSE).
