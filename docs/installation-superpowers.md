# superpowers — agentic workflow skills

> **Generic capability:** a skill pack for brainstorming, debugging, TDD, planning, and review that auto-loads per task.
>
> **This page's implementation:** generic. Installation differs per harness — each agent has its own plugin command — but the skills and the bootstrap are the same Markdown everywhere. Pick your agent from the table below.

> [superpowers](https://github.com/obra/superpowers) provides a curated skill set for brainstorming, debugging, TDD, planning, code review, and more. Skills auto-load per task — no manual invocation once installed.

## Install per harness

`cd` into the project directory first, then use the row for your agent:

| Harness | Command |
|---|---|
| Claude Code | `/plugin install superpowers@claude-plugins-official` (official marketplace) — or `/plugin marketplace add obra/superpowers-marketplace` then `/plugin install superpowers@superpowers-marketplace` |
| Cursor | `/add-plugin superpowers`, or search "superpowers" in the plugin marketplace |
| Codex CLI | `/plugins`, search `superpowers`, pick Install Plugin |
| OpenCode | ask the agent to *fetch and follow* `https://raw.githubusercontent.com/obra/superpowers/refs/heads/main/.opencode/INSTALL.md` |
| Gemini CLI | `gemini extensions install https://github.com/obra/superpowers` |

The rest of the harnesses (Copilot CLI, Factory Droid, Kimi, Qwen, Muse, …) have their own plugin command — see the upstream README's Installation section. If you use more than one agent, install superpowers **separately for each**.

This installs the skills plus a small bootstrap that injects the `using-superpowers` instructions at session start. The agent auto-discovers them on next startup.

## Optional: subagent support

A few superpowers workflows (delegated execution, code review) need a `subagent` tool. Some harnesses ship one; where yours doesn't, either add its subagent plugin or skip those workflows — the skill content is unaffected.

Skip if you only plan to use the lighter skills (brainstorming, TDD, planning).

## Usage

Skills auto-load per task. The agent decides which skill to invoke based on the user's request. You can also trigger explicitly with `/skill:<name>` once a session is running.

## Recommendation: install at the project level

- Skills are scoped to the project that benefits from them
- Keeps unrelated projects free of skill suggestions
- Matches how superpowers is intended (per-project, not per-machine)

## Avoid: global install

Global install loads superpowers on every session in every project, regardless of fit. Not recommended.

## Scope

Superpowers only changes **what the agent knows** (skill content) and **how the bootstrap flows** (when skills are suggested). It does not modify the agent's built-in tools, model selection, or theme.
