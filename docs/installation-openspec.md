# OpenSpec — spec-driven development

> **Generic capability:** turn a change into a spec, then into tasks, then into code, with an archive step — a workflow shape, not a runtime feature.
>
> **This page's implementation:** `openspec init --tools <name>`. OpenSpec targets several agents; swap the `--tools` value (or pick from the interactive list) to write its skills and commands for yours.

> [OpenSpec](https://github.com/Fission-AI/OpenSpec) is a spec-driven development tool for generating and managing project specifications. It ships integrations for several agents — pick yours during `openspec init`.

## Prerequisites

```bash
which openspec || npm install -g @fission-ai/openspec@latest
```

## One-line init

`cd` into the project directory, then run:

```bash
openspec init --tools <your-agent>
```

This bootstraps OpenSpec in your project with the selected tool integration. Skills land in `$SKILLS_DIR/openspec-*/` and slash commands wherever your agent reads them (`$AGENT_HOME/prompts/`, `.claude/commands/`, …).

For interactive tool selection (also works):

```bash
openspec init
```

…then choose your agent from the list.

## Usage

After init, these slash commands are available in any session:

| Command | Purpose |
|---|---|
| `/opsx:propose` | Start a new change proposal |
| `/opsx:apply` | Apply a previously approved change |
| `/opsx:archive` | Archive a completed change |
| `/opsx:sync` | Sync specs to match code changes |

## Recommendation: install at the project level

- Specs are project-specific — global install adds unnecessary context to every project
- Keeps the agent focused on the specs that matter for the current project
- Matches the intended usage pattern (per-project, not per-machine)

## Avoid: global install

Global install loads spec tools into every session, cluttering the agent's context. Not recommended.

## Scope

OpenSpec only writes project-local files (`$SKILLS_DIR/openspec-*/SKILL.md` plus the matching command files). It does not touch your global `$AGENT_HOME`, model settings, or other projects.