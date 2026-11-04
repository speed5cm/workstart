# codegraph — pre-indexed code graph

> **Generic capability:** pre-index the codebase into a graph so the agent gets call chains and blast radius in one query instead of crawling files.
>
> **This page's implementation:** generic. The installer needs *some* target to generate `AGENTS.md`, so the guide uses the `opencode` target and deletes the leftover opencode config — the MCP server and the graph itself are runtime-neutral.

> [codegraph](https://github.com/colbymchenry/codegraph) pre-indexes the codebase into a knowledge graph. The agent gets precise context (call chains, blast radius, related symbols) in one query — instead of crawling files one by one.

## Install codegraph

The installer wants a target to write agent instructions for. `opencode` is used here because it emits a portable `AGENTS.md`; any target that does the same works — only the leftover config file name changes:

```bash
npx -y @colbymchenry/codegraph install -t opencode -l local -y
```

This installs the codegraph binary and writes two files in the project directory:

- `AGENTS.md` — agent instructions for using codegraph (kept, works for every agent)
- `opencode.jsonc` — opencode MCP config (deleted below, since only `AGENTS.md` is wanted)

Delete `opencode.jsonc` immediately after install — it is only needed if you actually run opencode:

```bash
rm opencode.jsonc
```

Then build the per-project graph:

```bash
npx -y @colbymchenry/codegraph init
```

All steps are required: `install` installs codegraph and updates `AGENTS.md`, deleting `opencode.jsonc` removes the unused opencode config, the MCP entry below connects codegraph to the agent, and `init` builds the index. Without `init`, the MCP tools have nothing to query.

## Configure MCP

Create or update your agent's MCP config in the project root — `.mcp.json` is the common name; use whatever path your agent reads. If the file already exists, merge the `codegraph` entry into its existing `mcpServers` object instead of replacing it.

```json
{
  "mcpServers": {
    "codegraph": {
      "command": "codegraph",
      "args": ["serve", "--mcp"]
    }
  }
}
```

Restart the agent. In the MCP listing (`/mcp` in Claude Code, or the equivalent) `codegraph` should appear.

## Verify

In a fresh session:

> "How does function X reach function Y?"

The agent should answer using the codegraph tools (you'll see it call `codegraph_explore`), returning call paths and source snippets rather than a series of `grep` results.

## Recommendation: install at the project level

- The graph index lives in `.codegraph/` — keeps it alongside the code it describes
- Unrelated projects stay unaffected
- Matches how codegraph is intended to be used (per-project, not per-machine)

## Scope

codegraph only exposes graph-query tools. It does not modify the agent's other MCP servers, model selection, or theme. The index is regenerated from source — add `.codegraph/` to your project's `.gitignore`.