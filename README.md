# Vibe Variables

A free naming convention for bracket style placeholders, like `{Username}` or `{Today's Date}`, that tell an AI coding agent exactly what a placeholder in a design should become.

This isn't a new framework or a library. It's a way of writing placeholder text in a design so that any AI reading it, whether through Figma, a screenshot, or a hand drawn wireframe, knows it's supposed to become real, dynamic data instead of static copy.

## The patterns

| Pattern | Example | Meaning |
|---|---|---|
| Plain variable | `{Username}` | A bracket with just a name. No example value, no state. |
| Variable + state | `{Today's Date}{Hover-state}` | The second bracket describes a look, not a new variable. |
| Example + variable | `John Doe{username}` | Text glued to a bracket with no space. The text is the real placeholder value. |
| Computed variable | `{Profit}` | A result derived from other variables, like Price and Margin. Never its own input. |
| Value in bracket | `{20%}` | The value sits inside the bracket itself. A nearby label supplies the name. |

## Using it

This repo contains a single `SKILL.md` file written for Claude's [Agent Skills](https://www.anthropic.com/news/skills) format, an open spec also published at [agentskills.io](https://agentskills.io), so it isn't locked to one tool.

**Claude (claude.ai, Claude Code, or the API):** upload `SKILL.md` as a custom skill, then invoke it directly with `/vibe-variables`, or just hand over a design with curly braces in it and let Claude pick it up on its own.

**Cursor, Windsurf, or similar:** drop `SKILL.md` into your project's rules or context folder and reference it with `@SKILL.md` when prompting.

**Any other AI tool:** paste the contents of `SKILL.md` into your system prompt or first message, then hand over the design as usual.

## License

Free to use, including commercially. Credit is appreciated, not required.

---

Built by [@design.ustaad](https://www.instagram.com/design.ustaad/).
