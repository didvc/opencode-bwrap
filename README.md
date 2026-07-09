# opencode-bwrap

Run [opencode](https://github.com/anomalyco/opencode) inside a [bubblewrap](https://github.com/containers/bubblewrap) sandbox — keeping AI-driven file operations confined to your current working directory.

> **See also: [`ai-bwrap`](https://github.com/didvc/ai-bwrap)** — the multi-agent sequel to this project. Same sandbox approach, extended to Claude Code, opencode, Grok, or a plain shell, with an extensible agent registry.

## What it does

`opencode-bwrap` is a thin shell wrapper that launches opencode inside a `bwrap` (bubblewrap) sandbox. The sandbox:

- Gives opencode **read-write access only to your current working directory**
- Blocks access to the rest of `$HOME` (no `~/.ssh`, `~/Documents`, etc.)
- Keeps network access open (required for LLM API calls)
- Isolates PID, IPC, UTS, and mount namespaces
- Passes through your opencode config, cache, and state directories
- Passes through your `~/.gitconfig` and `gh` CLI auth (read-only)
- Supports NVM-managed Node.js installations

## Requirements

- Linux
- [`bwrap`](https://github.com/containers/bubblewrap) — install via your package manager:
  ```sh
  # Debian/Ubuntu
  sudo apt install bubblewrap

  # Arch
  sudo pacman -S bubblewrap

  # Fedora
  sudo dnf install bubblewrap
  ```
- [opencode](https://opencode.ai) installed and on `$PATH`

## Installation

1. Copy `opencode-bwrap` to somewhere on your `$PATH`, for example:
   ```sh
   cp opencode-bwrap ~/.local/bin/opencode-bwrap
   chmod +x ~/.local/bin/opencode-bwrap
   ```

2. Run it from your project directory:
   ```sh
   cd /path/to/your/project
   opencode-bwrap
   ```

   Pass any opencode flags normally:
   ```sh
   opencode-bwrap --model anthropic/claude-sonnet-4-5
   ```

## What the sandbox blocks

| Resource | Behavior |
|---|---|
| Files outside `$PWD` | Not visible (no bind) |
| `~/.ssh` | Not mounted |
| `$HOME` (other than config dirs) | Empty virtual dir |
| Physical block devices (`/dev/sda`, etc.) | Not present |
| Host processes (other PIDs) | Not visible — isolated PID namespace |
| Privilege escalation (`sudo`) | Blocked — no `setuid` in namespace |

## What is passed through (read-only unless noted)

| Resource | Mount type |
|---|---|
| `$PWD` (current directory) | **read-write** |
| `~/.config/opencode` | read-write |
| `~/.local/share/opencode` | read-write |
| `~/.cache/opencode` | read-write |
| `~/.local/state/opencode` | read-write |
| `~/.gitconfig` | read-only |
| `~/.config/gh` | read-only |
| `$NVM_DIR` | read-only |
| `/usr`, `/etc` | read-only |

## Adding more workspaces

Edit the `WORKSPACES` array at the top of the script:

```sh
WORKSPACES=(
    "$(pwd)"
    "$HOME/projects/other-repo"
)
```

## Security notes

`bwrap` uses Linux namespaces — the same kernel is shared with the host. This is stronger isolation than running opencode directly, and lighter than a full VM. For most personal development use cases this is a practical tradeoff.

Key limitations to be aware of:
- The host kernel is shared. A kernel-level exploit could escape the sandbox.
- `--new-session` is intentionally omitted to preserve job control (`Ctrl-Z` / `fg`). This slightly reduces TIOCSTI protection.
- Anything explicitly bind-mounted is accessible to the sandboxed process.

## License

MIT

<!-- BEGIN gh-mutual-linking -->

---

### Related projects

- [**ai-bwrap**](https://github.com/didvc/ai-bwrap) — Run AI coding agents (Claude Code, opencode, Grok, ...) inside a bubblewrap sandbox — one wrapper, any agent.
- [**agent.txtar**](https://github.com/didvc/agent.txtar) — Portable text block for giving structured context to AI agents — embed anywhere, parse anywhere
- [**claude-code-jsonl-editor**](https://github.com/didvc/claude-code-jsonl-editor) — 🚀 Interactive JSONL editor for Claude Code conversation files with real-time file system synchronization. Efficient prompt engineering through conversation editing.
<!-- END gh-mutual-linking -->