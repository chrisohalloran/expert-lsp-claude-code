# elixir-expert-lsp — Elixir Code Intelligence for Claude Code

A Claude Code plugin that connects [Expert LSP](https://expert-lsp.org) — the official unified Elixir language server — to give Claude real-time code intelligence while working on Elixir projects.

## What Claude gains

- **Auto-diagnostics**: After every file edit, Expert analyzes changes and reports errors, warnings, and missing imports. Claude sees issues immediately and fixes them in the same turn.
- **Code navigation**: Go to definition, find references, hover for type info, list symbols, code actions — precise navigation instead of grep-based search.

### Verified capabilities

| Capability | Status |
|-----------|--------|
| File change sync | Yes |
| Code completion | Yes |
| Hover / type info | Yes |
| Go to definition | Yes |
| Find references | Yes |
| Document symbols | Yes |
| Workspace symbols | Yes |
| Code actions (quick fixes) | Yes |
| Code lens | Yes |
| Formatting | Yes |
| Execute commands | Yes |

## Benchmarks (vs ElixirLS)

Tested against a real Elixir/Phoenix project (Rig, 8 lib files):

| Metric | Expert LSP | ElixirLS |
|--------|-----------|----------|
| Project compile | 12s (513ms engine) | 30s |
| Diagnostic latency | **1.0s** | No diagnostics |
| Error detection | Caught `undefined function run/3` | Missed it |
| Incremental recompile | 58ms | N/A |

Expert compiles 2.5x faster and actually catches errors that ElixirLS misses.

## First Run

On first use per project, Expert builds an analysis engine (~30s). This is cached at `~/Library/Caches/expert/` and subsequent starts take <1s. You can warm the cache by running `expert --stdio` in your project directory and waiting for it to finish.

## Requirements

- **Claude Code** v1.0.33+
- **Expert LSP** v0.1.0-rc.6+ installed and in your `$PATH`
- **Elixir** 1.15+ with OTP 26+

## Install the plugin

```bash
# From the Claude Code CLI
claude plugin install elixir-expert-lsp@<marketplace>

# Or load locally for development
claude --plugin-dir /path/to/expert-lsp
```

## Install Expert LSP

The plugin configures the connection but doesn't include the server binary. Install Expert using one of these methods:

### Download pre-built binary (recommended)

Download the latest release for your platform from [github.com/elixir-lang/expert/releases](https://github.com/elixir-lang/expert/releases) and place it in your `$PATH`:

```bash
# macOS ARM (Apple Silicon)
gh release download v0.1.0-rc.6 --pattern 'expert_darwin_arm64' --repo elixir-lang/expert --dir /tmp
cp /tmp/expert_darwin_arm64 ~/.local/bin/expert
chmod +x ~/.local/bin/expert

# macOS Intel
gh release download v0.1.0-rc.6 --pattern 'expert_darwin_amd64' --repo elixir-lang/expert --dir /tmp
cp /tmp/expert_darwin_amd64 ~/.local/bin/expert
chmod +x ~/.local/bin/expert

# Linux
gh release download v0.1.0-rc.6 --pattern 'expert_linux_amd64' --repo elixir-lang/expert --dir /tmp
cp /tmp/expert_linux_amd64 ~/.local/bin/expert
chmod +x ~/.local/bin/expert
```

### Nightly builds

```bash
gh release download nightly --pattern 'expert_darwin_arm64' --repo elixir-lang/expert --dir /tmp
cp /tmp/expert_darwin_arm64 ~/.local/bin/expert
chmod +x ~/.local/bin/expert
```

### Verify installation

```bash
expert --version
# Should output: 0.1.0-rc.6 (or newer)
```

Make sure `~/.local/bin` is in your `$PATH`. Add this to your `~/.zshrc` if needed:

```bash
export PATH="$HOME/.local/bin:$PATH"
```

## Supported file types

| Extension | Language ID |
|-----------|------------|
| `.ex`     | elixir     |
| `.exs`    | elixir     |
| `.heex`   | elixir     |

## How it works

This plugin provides an `.lsp.json` configuration that tells Claude Code how to connect to Expert via stdio transport. Expert runs as a subprocess spawned by Claude Code, analyzing your project files and providing real-time diagnostics and navigation.

When you edit an Elixir file, Expert analyzes the changes and pushes diagnostics back to Claude Code. Claude sees type errors, missing imports, and compilation warnings immediately and can fix them in the same turn — without you running `mix compile` manually.

## Expert LSP vs ElixirLS

Expert is the [official unified Elixir language server](https://elixir-lang.org/blog/2024/08/15/welcome-elixir-language-server-team/), created by the Elixir core team by merging the three previous implementations (ElixirLS, Lexical, Next LS). It's built on the Lexical codebase with Next LS components for single-binary releases and protocol abstraction.

Expert is currently in release candidate status (v0.1.0-rc.6 as of March 2026). If you encounter issues, you can fall back to the `elixir-ls-lsp` plugin which uses the legacy ElixirLS server:

```bash
claude plugin disable elixir-expert-lsp
claude plugin enable elixir-ls-lsp@claude-plugins-official
```

## Configuration

The plugin uses sensible defaults:
- Transport: stdio (`expert --stdio`)
- Auto-restart on crash: up to 3 times
- No additional initialization options required

## Troubleshooting

**"Executable not found in $PATH"**: Install Expert using the instructions above and ensure `~/.local/bin` is in your PATH.

**Plugin not loading**: Run `/plugin` in Claude Code and check the Errors tab. If Expert crashes on startup, try the nightly build which may have fixes.

**Conflicts with elixir-ls-lsp**: Disable one before enabling the other. Both configure an LSP for `.ex` files and may conflict.

## License

MIT
