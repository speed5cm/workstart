# caveman — compressed prose output

> **Generic capability:** compress the agent's natural-language output (~65–75%) while keeping technical accuracy.
>
> **This page's implementation:** Pi (`.pi/skills/`). The installer's `--agent` flag is the runtime switch — point it at a different agent and the skill lands in that agent's directory instead.

> [caveman](https://github.com/juliusbrussee/caveman) is an ultra-compressed communication mode that cuts token usage ~75% while preserving technical accuracy. Six intensity levels, triggered by user keywords like "caveman mode" or `/caveman`.

## One-line install

`cd` into the project directory, then run:

```
npx skills add https://github.com/juliusbrussee/caveman --skill caveman --agent pi -y
```

This installs the skill into the project's skill directory — with `--agent pi` that resolves to `.pi/skills/`.

## Usage

Once installed, trigger with `/caveman` or keywords like "caveman mode" or "less tokens". Disable with "stop caveman" or "normal mode".

## Recommendation: install at the project level

- Reduces token costs for the projects that benefit from compressed output
- Keeps default verbose behavior in projects that don't need it
- Simple to toggle per project

## Avoid: global install

Global install applies compression to every session, including ones where verbose output is preferred. Not recommended.