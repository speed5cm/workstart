# hallmark — design skill that refuses to look AI-generated

> **Generic capability:** a design skill that holds a hard floor (contrast, focus, reduced motion, no invented facts), deals a direction instead of converging on the same defaults every time, and lints the result in a real browser. It is Markdown + Node scripts, not code bound to a runtime.
>
> **This page's implementation:** the `skills` CLI. `npx skills add` copies the skill folder into whichever harness directory you target — Claude Code reads `~/.claude/skills/`, Codex and OpenCode read `~/.agents/skills/` — and `sloplint` runs as an ordinary Node script. No runtime-specific glue is involved.

> [hallmark](https://github.com/Nutlope/hallmark) — **Powered by Together AI** — is a design skill for Claude Code, Codex, Cursor and OpenCode that refuses to look AI-generated. It deals three themes from a catalog of fifty materials, builds the page around what the visitor came to do, and lints the result against the tells that read as machine-made: the eyebrow above every heading, tracked-capital labels, one coloured word in a headline, a page of empty frames or nothing but photographs, a hero that misses its screen, an email sign-up at the end of every page, a form that says thanks without sending anything.

## What it actually does

The gates come in three tiers, and the tier is the point:

| Tier | Behaviour |
|---|---|
| **Floor** | Never ships. Contrast, focus, reduced motion, invented metrics or testimonials, a broken phone layout, `transition: all`, `overflow-x: hidden` on `html`/`body`, an emoji used as an icon. Not waivable. |
| **Reflex** | The defaults a model falls into. A build with a real reason may overrule one **on the record** — the waiver is written into the CSS stamp. |
| **Finish** | Craft detail that never blocks a ship. Answered, not enforced. |

A build runs in a fixed order: pre-flight reads the project (fonts, palette, framework, a locked `design.md`), asks **once** for the material (photos, screenshots, real numbers — nothing is invented), then `theme.mjs deal` offers three themes that fit the prompt, the hero is built alone and reviewed cold, and `sloplint.mjs --render` checks the page at eight widths before a fresh-context reviewer gets the last word.

Every page is self-contained HTML and CSS, stamped in its first CSS comment with the decisions it took.

## Requirements

- **Node 18 or later** — `node --version` to confirm.
- For the browser checks (`sloplint --render`, screenshots): `npm i -g puppeteer-core` and Chrome installed. Without them the static checks still run; the rendered ones are skipped, which is where most of the real value is.
- No API key. The photo catalog (755 openly licensed photos, picked per subject and credited on the page) ships with the skill.

## One-line install

`cd` into the project directory, then run:

```bash
npx skills add nutlope/hallmark -y
```

Without `--agent` it installs for every supported harness, into the current project; pass `-g` to install for every project instead. The agent picks the skill up on next startup — nothing to invoke.

> The upstream README lists five targets: `claude-code`, `codex`, `opencode`, `cursor` and `github-copilot`. Narrow it with `--agent` (see below) when you only want one.

### Where it lands, per harness

| Harness | Directory | Invoke |
|---|---|---|
| Claude Code | `~/.claude/skills/hallmark` | `/hallmark <prompt>`, or describe a new build |
| Codex CLI | `~/.agents/skills/hallmark` | `$hallmark <prompt>`, or describe a new build |
| Cursor | finds the copies above | `/hallmark <prompt>` |
| OpenCode | finds the copies above | `/hallmark <prompt>` |

### One harness only

```bash
npx skills add nutlope/hallmark -g --agent claude-code
```

Swap `claude-code` for `codex`, `opencode`, `cursor` or `github-copilot`.

### As a Claude Code plugin

```
/plugin marketplace add nutlope/hallmark
/plugin install hallmark@hallmark
```

Invoked as `/hallmark:hallmark`. The plugin ships only the skill, not the whole repo.

### From a clone

```bash
git clone https://github.com/Nutlope/hallmark
node hallmark/skills/hallmark/scripts/install.mjs
```

`install.mjs` validates every theme and reference before it copies anything. `node hallmark/skills/hallmark/scripts/install-hook.mjs` additionally adds an edit-time Floor check to Claude Code.

## Usage

Three shapes, and only the first is what you get by default:

| Shape | Trigger | What happens |
|---|---|---|
| **New build** | describe what you want, or `/hallmark <prompt>` | Full run: pre-flight, one question, theme deal, hero first, then the page |
| **Redesign** | `/hallmark redesign <path>` | Rebuilds the page, keeps its content and wiring. Say *keep the layout* for a restyle, *polish* for only the tells |
| **Audit** | `/hallmark audit <path>` | Report only, no edits |

**Hallmark never touches an existing design unless you name it.** Any other change to a live design is not a Hallmark run — the agent should do it as asked, in the project's own system.

`sloplint --before` records every link, field, handler and heading first; `sloplint --kept` fails the redesign if one went missing. A redesign is expected to hand back a side-by-side before and after.

If it does not fire on its own, name it: *use hallmark to build ...*

## Lint any page

`sloplint` works standalone — you don't need a Hallmark build to run it:

```bash
node "$SKILLS_DIR/hallmark/scripts/sloplint.mjs" <file-or-dir> --render
```

Fix every `FAIL`. Fix or waive every `REFLEX` — a waiver needs a real reason and belongs in the CSS stamp next to the floor line.

## Recommendation: install at the project level

- A locked `design.md` is the input to pre-flight; it lives with the project, and a global install would read the wrong one
- Themes get chosen against the project's fonts and palette — a global deal has nothing to read
- The delivered artefact is self-contained HTML/CSS in the repo, so review happens in the diff
- Keeps unrelated projects free of a design skill that would otherwise offer itself on every build

## Avoid: global install

`-g` puts the skill in every session in every project, including the ones where you are editing a WebForms `.aspx` or debugging a database and a 7 MB theme catalog is dead weight. It also means the deal runs without a project to read.

## Updating

```bash
npx skills update
```

Updates whatever is installed at the current scope. The repo moves fast — v2 added fifty themes, the custom path, real product screenshots and the redesign/audit verbs.

## Scope

hallmark writes **project-local files only**:

- `$SKILLS_DIR/hallmark/` — `SKILL.md`, `references/` (themes, genres, gates, craft), `scripts/` (`sloplint`, `theme`, `capture`, `picture`), and the photo catalog
- During a build: the page's own HTML/CSS, a `tokens.css`, the CSS stamp, and the build log
- Nothing else. It does not touch your global agent config, model settings, theme, or other projects.

Uninstall with `npx skills remove hallmark`, or delete the directory.
