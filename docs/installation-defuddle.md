# defuddle — page content as clean Markdown

> **Generic capability:** strip a web page down to its main content and hand the agent Markdown instead of the whole page.
>
> **This page's implementation:** a standalone npm CLI (`defuddle`), invoked by the agent directly. No MCP server, no host hook — any agent that can run a shell command can use it.

> [defuddle](https://github.com/kepano/defuddle) (MIT, by Steph Ango) takes a URL or HTML, finds the main content, and returns cleaned HTML or Markdown. It was built as the extraction engine behind Obsidian Web Clipper and is positioned as a more forgiving replacement for Mozilla Readability.

A raw fetch gives the agent the article plus everything around it: navigation, cookie banners, ads, related links, comment threads, newsletter prompts, footers. That noise is paid for in context, and then paid for again while the agent works out what to ignore — usually badly. defuddle removes it before the text ever reaches the agent.

## Install in the project

`cd` into the project, then:

```bash
npm install -D defuddle
```

Everything lands in that project's `node_modules` and lockfile; nothing touches the global npm prefix. Node.js and npm are required.

Call it through `npx`, which resolves the local binary:

```bash
npx defuddle parse https://example.com/article --md
```

Running `npx defuddle` without a local install downloads the latest version on every call — fine for a one-off look, not for a workflow that runs on every page.

## Usage

```
npx defuddle parse <url | file | -> [options]
```

```bash
# A URL, straight to Markdown
npx defuddle parse https://example.com/article --md

# Markdown into a file
npx defuddle parse https://example.com/article --md -o article.md

# Only one metadata field, for a script
npx defuddle parse https://example.com/article -p title

# HTML you already fetched
curl -L https://example.com/article | npx defuddle parse --md
```

The useful options:

| Option | Alias | What it does |
|---|---|---|
| `--markdown` | `-m`, `--md` | Output Markdown instead of cleaned HTML |
| `--json` | `-j` | Output content plus metadata as JSON |
| `--frontmatter` | `-f` | Prepend YAML frontmatter (title, author, source, …) |
| `--property <name>` | `-p` | Print a single field — `title`, `description`, `domain`, … |
| `--output <file>` | `-o` | Write to a file instead of stdout |
| `--user-agent <string>` | `-u` | Custom User-Agent for sites that answer 403 |
| `--debug` | | Show which element was chosen as content and what was removed |

## What survives extraction

The article body, plus metadata: `title`, `author`, `description`, `published`, `site`, `domain`, `language`, `image`, `wordCount`. Headings are normalized, code blocks keep their language (line numbers and syntax-highlight colors are dropped), footnotes and math are converted to standard formats, and callouts become Obsidian-style blockquotes in Markdown.

## Recommendation: install at the project level

- Only the projects that actually read the web pay for it; the rest of the machine stays clean
- The lockfile pins the extraction version, so output does not shift under you when the library ships a new heuristic
- The trade-off is path-bound: the agent has to run from that project's directory for `npx` to find the local binary

## Avoid: global install

`npm install -g defuddle` puts one version on every project and upgrades it silently. The project's own README opens with "Beware! Defuddle is very much a work in progress" — unannounced changes are exactly what you don't want in something that feeds text straight into context.

## Wire it into the agent

Add one line to the project's `AGENTS.md`:

```markdown
- Fetch web pages with `npx defuddle parse <url> --md` instead of a plain fetch; it drops navigation, ads and clutter before the text enters context.
```

The line goes with the project that installed the dependency — a global `AGENTS.md` would advertise a command most projects do not have on disk.

## Caveats

- The project describes itself as "very much a work in progress" — expect occasional extraction misses on unusual layouts.
- Sites that render their content client-side can come back thin; check before trusting the output.
- Some sites reject the default User-Agent. Pass `--user-agent` with a normal browser string.
- Pinning a selector (`contentSelector` in the library) is the escape hatch when auto-detection picks the wrong block. The CLI covers the common cases; the library is where the fine control lives.
