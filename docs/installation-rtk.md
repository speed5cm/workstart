# rtk — compact shell output

> **Generic capability:** transparently rewrite verbose shell commands into a token-cheap form. Any agent that shells out benefits; the saving is in the output, not the agent.
>
> **This page's implementation:** Pi. `--agent pi` makes rtk emit an extension at `~/.pi/agent/extensions/rtk.ts`. Another runtime needs an equivalent interception point (hook, wrapper, or shell proxy) — the *global, not per-project* call carries over unchanged.

[rtk](https://github.com/rtk-ai/rtk) transparently rewrites verbose shell commands (`git status`, `pnpm list`, `vitest`, `cargo test`, …) into compact token-saving form. Once installed, each agent session is intercepted and rewritten automatically — same workflow, fewer tokens.

## Install

```bash
rtk init --agent pi --global
```

`--agent pi` picks the host: the command writes `~/.pi/agent/extensions/rtk.ts`, and the agent picks it up on startup. Pass another runtime's name to target that one instead.

> Install at the **global** level: rtk is most useful when every project's session benefits from token-saved bash output. A single global install covers all work without per-project setup.

## Verify

```bash
rtk init --show
```

Then in a fresh session, run a chatty command like `git status` and confirm the agent sees a shorter (rewritten) version instead of the raw output.

## Activate

Restart the agent, or run `/reload` inside an existing session.

## Uninstall

```bash
rtk init --uninstall --agent pi --global
```

Removes only the installed extension file. Other files in `~/.pi/agent/extensions/` are untouched.

## Scope

rtk currently only wraps the `bash` tool call. File reads (`read` / `grep` / `find` / `ls`), `write`, and `edit` are not rewritten.
