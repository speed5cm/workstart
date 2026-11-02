# rtk — compact shell output

> **Generic capability:** rewrite verbose shell commands and their output into a token-cheap form before the agent reads them. Any agent that shells out benefits; the saving is in the output, not the agent.
>
> **This page's implementation:** `rtk init -g --agent pi` — a TypeScript extension at `~/.pi/agent/extensions/rtk.ts`. rtk ships first-class targets for 18 agents (Claude Code, Cursor, Codex, Gemini CLI, Windsurf, Cline, OpenCode, …); only the `--agent` value changes.

[rtk](https://github.com/rtk-ai/rtk) is a single Rust binary that intercepts shell commands and compresses their output before the agent reads it — 125+ commands covered, under 20 ms overhead, ~56% smaller output on average and up to 90% on the noisiest ones.

## 1. Install the binary

```bash
brew install rtk-ai/tap/rtk                    # macOS / Linux
winget install rtk-ai.rtk                      # Windows
curl -fsSL https://raw.githubusercontent.com/rtk-ai/rtk/refs/heads/master/install.sh | sh
```

> **Name collision:** an unrelated project called `rtk` (Rust Type Kit) exists. `rtk gain` must print a savings dashboard — if it says "command not found", you installed the wrong one.

## 2. Wire it into your agent

```bash
rtk init -g                  # global; default target is Claude Code / Copilot
rtk init -g --agent pi       # global, for Pi
rtk init -g --agent cursor   # global, for Cursor
```

`-g` / `--global` installs a hook that applies to every project — this is what rewrites commands.

Without `-g`, `rtk init` only appends a rule block to `./CLAUDE.md` and drops a `.rtk/filters.toml` template: **no hook, nothing gets rewritten.** Don't use that form for this guide.

Other documented targets: `--gemini` (Gemini CLI), `--codex` (Codex), `--copilot`, `--opencode`, and `--agent windsurf` / `cline` / `kilocode` / `antigravity` / `kimi` / `omp` / `hermes` / `droid` / `trae` / `vibe`.

## 3. Verify

```bash
rtk --version
rtk gain
```

Then in a fresh session run a chatty command like `git status` — the agent should see the compact form, without ever calling `rtk` itself.

## 4. Activate

Restart the agent, or run `/reload` inside an existing session.

## Scope

The hook only fires on **shell tool calls**. Built-in tools (`Read`, `Grep`, `Glob`) bypass it entirely — to compress those, go through the shell (`cat`, `rg`, `find`) or call `rtk read` / `rtk grep` / `rtk find` directly.

## Uninstall

```bash
rtk init -g --uninstall      # removes the hook, RTK.md and the settings entry
```

If you installed with `--agent <name>`, pass the same `--agent` here, and add `--dry-run` first to see what would be removed.
