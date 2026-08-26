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

### Related projects

- [**lpchart**](https://github.com/didvc/lpchart) — Chart InfluxDB line protocol in your terminal. Browse measurements, fields and tag sets interactively without knowing what is in the file first.
- [**claude-chrome-multi-instance-warn**](https://github.com/didvc/claude-chrome-multi-instance-warn) — Claude Code plugin: warns once per session when a tool payload is about to launch another browser (Chrome/Playwright/Puppeteer/CDP) or fan work…
- [**histbak**](https://github.com/didvc/histbak) — Scheduled, compressed, optionally encrypted backups of your browsing history, plus a viewer that makes it readable. Local only, no network access…
- [**claude-email-leak-notice**](https://github.com/didvc/claude-email-leak-notice) — Claude Code plugin: warns once per session when a personal email address appears in a tool payload, so it does not end up in a commit or a pushed…
- [**nimo**](https://github.com/didvc/nimo) — nano-inspired terminal text editor written in pure Nim.
- [**terminal-scheduler**](https://github.com/vsc-ext/terminal-scheduler) — Prompt automation for terminal-based agents, for VSCode.
- [**screen-masking**](https://github.com/didvc/screen-masking) — Cover parts of your Windows desktop with non-interactive overlays you shape from a pixel-ruled preview window. Pure Win32, no dependencies.
- [**dead-mans-ping**](https://github.com/didvc/dead-mans-ping) — Cross-platform CLI that HTTP-GET pings endpoints on mouse (in)activity — dead-man's switch, presence beacon, idle notifier.
- [**ai-bwrap**](https://github.com/didvc/ai-bwrap) — Run AI coding agents (Claude Code, opencode, Grok, ...) inside a bubblewrap sandbox — one wrapper, any agent.
- [**simple-ots**](https://github.com/didvc/simple-ots) — Hash files, build a Merkle tree, anchor to Bitcoin via OpenTimestamps. Selective disclosure without ZKP.
- [**agent.txtar**](https://github.com/didvc/agent.txtar) — Portable text block for giving structured context to AI agents — embed anywhere, parse anywhere
- [**http-status-monitor**](https://github.com/didvc/http-status-monitor) — CLI that runs lychee against a URL list and tracks HTTP status/asset changes over time
- [**git-dup**](https://github.com/didvc/git-dup) — Snapshot a git subdirectory (committed state + untracked files) into a timestamped sibling directory
- [**dir-cpu**](https://github.com/didvc/dir-cpu) — Real-time CLI that shows CPU usage aggregated by filesystem directory
- [**cf-cache-utils**](https://github.com/didvc/cf-cache-utils) — CLI to warm and inspect Cloudflare edge cache status across all your URLs — no external dependencies, pure Node.js
- [**claude-code-jsonl-editor**](https://github.com/didvc/claude-code-jsonl-editor) — 🚀 Interactive JSONL editor for Claude Code conversation files with real-time file system synchronization. Efficient prompt engineering through…
- [**mva**](https://github.com/didvc/mva) — mva (mv-archive) - rclone backup/archiving simpler, efficient, and graceful. Just mv files to trigger automatic compression and cloud upload.
- [**cli**](https://github.com/prompt-management/cli) — CLI tool for managing AI prompts with search and organization capabilities. Personal, secure prompt library with YAML storage and metadata filtering.
- [**cli**](https://github.com/prompt-scheduler/cli) — Modern TypeScript automation tool for scheduling and executing prompts for AI agents with intelligent usage limit detection. Currently supports…
- [**notification-cli**](https://github.com/didvc/notification-cli) — Minimalistic Command Line Notification Application under 50 Lines
- [**node-cron-cli**](https://github.com/didvc/node-cron-cli) — Command Line Implementation for Node Cron Under 20 Lines.
- [**note-cli**](https://github.com/didvc/note-cli) — Markdown Indexing and Pcre Regular Expression Compatible Full Text Searching for Advanced Note Takers.
<!-- END gh-mutual-linking -->