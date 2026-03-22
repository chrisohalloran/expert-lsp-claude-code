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

## Performance Notes

Tested against an Elixir project (Rig, 8 lib files). Results are from a single project and may vary based on project size, hardware, and Expert version:

| Metric | Expert LSP | ElixirLS v0.30.0 |
|--------|-----------|----------|
| Project compile | 12s | 30s |
| Diagnostic latency | 1.0s | Did not return diagnostics in test window |
| Incremental recompile | 58ms | N/A |

In this test, Expert detected an `undefined function run/3` error within 1 second of saving. ElixirLS did not return diagnostics within the test window. Expert is in release candidate status; ElixirLS is more mature. Your results will vary.

## First Run

On first use, Expert builds an analysis engine (~30 seconds). This is a one-time cost cached at `~/Library/Caches/expert/`. Subsequent starts take under 1 second.

To warm the cache before using in Claude Code:

```bash
cd your-project
expert --stdio
# Wait until you see "Engine initialized" in .expert/expert.log (~30s)
# Then Ctrl+C
```

Each project also needs an initial compile on first open (~5-10s with warm engine). After that, incremental recompiles take milliseconds.

## Requirements

- **Claude Code** v1.0.33+
- **Expert LSP** v0.1.0-rc.6+ installed and in your `$PATH`
- **Elixir** 1.15+ with OTP 26+ (check with `elixir --version`)

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
| `.ex`     | elixir       |
| `.exs`    | elixir       |
| `.heex`   | phoenix-heex |
| `.leex`   | elixir       |

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
- Startup timeout: 60 seconds (accommodates first-run engine build)
- Shutdown timeout: 5 seconds

## Troubleshooting

**"Executable not found in $PATH"**: Install Expert using the instructions above and ensure `~/.local/bin` is in your PATH. Verify with `expert --version`.

**No diagnostics appearing**: Expert needs to build its engine on first run (~30s). Check the log at `.expert/expert.log` in your project directory for progress. Look for "Engine initialized" to confirm it's ready.

**Plugin not loading**: Run `/plugin` in Claude Code and check the Errors tab. If Expert crashes on startup, try the nightly build which may have fixes:
```bash
gh release download nightly --pattern 'expert_darwin_arm64' --repo elixir-lang/expert --dir /tmp
cp /tmp/expert_darwin_arm64 ~/.local/bin/expert && chmod +x ~/.local/bin/expert
```

**Conflicts with elixir-ls-lsp**: Disable one before enabling the other. Both configure an LSP for `.ex` files and may conflict.

**Reporting issues**: Include your `expert --version`, `elixir --version`, and the contents of `.expert/expert.log` from your project directory.

## License

MIT
