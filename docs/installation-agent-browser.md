# agent-browser — browser control via CDP

> **Generic capability:** let the agent drive a real browser (open, click, fill, screenshot, read console/network) through Chrome DevTools Protocol.
>
> **This page's implementation:** the `skills` CLI. Only the target directory is runtime-specific (`$SKILLS_DIR`, e.g. `.claude/skills/` or `.agents/skills/`); the skill body and the CDP workflow are neutral.

> [agent-browser](https://github.com/vercel-labs/agent-browser) lets the agent control a browser via CDP (Chrome DevTools Protocol) — more token-efficient than traditional headless browser approaches.

## Prerequisites

```bash
which agent-browser || npm i -g agent-browser && agent-browser install
```

`agent-browser install` downloads the required Chromium binary.

## One-line install

`cd` into the project directory, then run:

```
npx skills add https://github.com/vercel-labs/agent-browser --skill agent-browser -y
```

This installs the skill into the project's skill directory. Add `--agent <name>` to target one harness (`claude-code`, `codex`, `cursor`, `opencode`, `github-copilot`); omit it to install for every supported one.

## Usage

In a fresh session, the agent can now drive the browser via the `agent-browser` skill — open pages, click elements, fill forms, take screenshots, and read console / network output. Ask the agent to do something on a real browser and it will invoke the skill.

## Recommendation: install at the project level

- Scopes the skill to projects that actually need browser automation
- Keeps the agent's tool list clean — only projects with web testing get the browser tools
- Matches the intended usage pattern (per-project, not per-machine)

## Avoid: global install

Global install adds browser tools to every project session, wasting tokens and context on projects that don't need it. Not recommended.