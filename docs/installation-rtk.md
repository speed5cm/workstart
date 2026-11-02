# rtk — compact output, optional routing

> **Generic capability:** rewrite verbose shell commands and their output into a token-cheap form before the agent reads them. Any agent that shells out benefits; the saving is in the output, not the agent. v1 optionally extends the same engine to provider-API traffic (compress payloads, route cheap calls to a cheaper model).
>
> **This page's implementation:** `rtk init -g --agent pi` — a TypeScript extension at `~/.pi/agent/extensions/rtk.ts`. rtk ships first-class targets for 18 agents (Claude Code, Cursor, Codex, Gemini CLI, Windsurf, Cline, OpenCode, …); only the `--agent` value changes. The optional proxy sits at the provider-API layer, so it is even less runtime-bound than the hook — but only `rtk claude` is wired for it today.

[rtk](https://github.com/rtk-ai/rtk) is a single Rust binary that intercepts shell commands and compresses their output before the agent reads it — 100+ commands covered, under 10 ms overhead, 60–90% smaller output on common commands and up to 90% on the noisiest ones.

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

```bash
rtk bypass <cmd>      # run one command unfiltered, still counted in `rtk gain`
```

## Optional: what v1 adds

rtk v1 is no longer only bash output — the same engine also runs a **pipeline** over the agent's provider-API traffic. Everything below is opt-in, one line at a time, and **upgrading changes nothing**: with no pipeline file you are on `rtk:default`, i.e. exactly the behaviour described above.

### `pipeline.toml` — behaviour as config

Which commands get rewritten, what happens to a tool's output, what the proxy compresses — all expressed in one composable file instead of being hard-coded:

```toml
# ~/.config/rtk/pipeline.toml
[pipeline]
extends = ["rtk:posthook", "rtk:proxy"]
```

### `rtk-proxy` — compress and route provider traffic

A separate daemon between the agent and the model provider: it compresses request payloads, and carries the calls your pipeline **routes to a cheaper model**. It borrows the credential the agent already sends, and its savings show up in `rtk gain` next to command filtering.

```bash
rtk claude                # start the daemon if needed, then launch the harness through it
rtk llm-proxy status      # is it up
rtk gain                  # command filtering + proxy side by side
```

Two ways in: `rtk init -g` offers to set it up once (writes the base URL and the `rtk:proxy` pipeline), or `rtk <harness>` needs no configuration and no hook system at all. `<harness>` currently resolves only for `rtk claude`; anything without a subcommand falls through to plain passthrough rather than being silently unproxied.

> **Licence split — read before wiring the proxy.** The proxy is a separate binary under **BUSL-1.1**, shipped as its own `rtk-proxy-<target>` archive; `rtk` never links it. The main binary also comes in two editions: the default (with the BUSL-1.1 compression engine) and `rtk-oss`, which is the hook rewrite system and every command filter under **Apache-2.0** alone — pick that one if your policy disallows source-available code in the build. Both binaries are named `rtk` and cannot be installed side by side. None of this affects steps 1–4 above.

## Uninstall

```bash
rtk init -g --uninstall      # removes the hook, RTK.md and the settings entry
```

If you installed with `--agent <name>`, pass the same `--agent` here, and add `--dry-run` first to see what would be removed.
