---
name: setup
description: Install the Expert LSP binary for the elixir-expert-lsp plugin
---

# Install Expert LSP

Check if Expert is installed, and install it if not.

## Steps

1. Check if `expert` is in PATH: `which expert`
2. If found, check version: `expert --version`
3. If NOT found, install via one of these methods:

### Download pre-built binary (recommended)
```bash
# macOS ARM (Apple Silicon)
gh release download v0.1.0-rc.6 --pattern 'expert_darwin_arm64' --repo elixir-lang/expert --dir /tmp
cp /tmp/expert_darwin_arm64 ~/.local/bin/expert && chmod +x ~/.local/bin/expert

# macOS Intel
gh release download v0.1.0-rc.6 --pattern 'expert_darwin_amd64' --repo elixir-lang/expert --dir /tmp
cp /tmp/expert_darwin_amd64 ~/.local/bin/expert && chmod +x ~/.local/bin/expert

# Linux
gh release download v0.1.0-rc.6 --pattern 'expert_linux_amd64' --repo elixir-lang/expert --dir /tmp
cp /tmp/expert_linux_amd64 ~/.local/bin/expert && chmod +x ~/.local/bin/expert
```

4. Verify: `expert --version`
5. Restart Claude Code to pick up the LSP server.
