<a href="https://nicktarasov.com">
  <img width="320" height="168" alt="skills" src="assets/skills.png" />
</a>

# skills

A small collection of [agent skills](https://code.claude.com/docs/en/skills) I use with Claude Code, published as reusable, brand-agnostic templates. Each skill is a self-contained folder — a `SKILL.md` plus supporting reference files — that you can drop into your own agent setup and adapt.

## Install

```bash
npx skills add nickdraft/skills
```

Or add a single skill manually by cloning its folder into your agents directory:

```bash
git clone https://github.com/nickdraft/skills
cp -R skills/skills/<name> ~/.claude/skills/<name>
```

## Skills

| Skill | What it does |
|---|---|
| [linear-pr-merge-sync](skills/linear-pr-merge-sync/SKILL.md) | Reconciles Linear issues, sub-issues, and project status against a PR (open or merged) — comments what shipped, flags what's missing, and files follow-up issues, all gated on your approval. |
| [linear-prioritise-issues](skills/linear-prioritise-issues/SKILL.md) | Wires an explicit execution order across a set of Linear issues using blocking relations + priority bumps, so the sequence reads at a glance in list views. |
| [design-component](skills/design-component/SKILL.md) | Design-system workflow for building UI components: Figma → build (CVA, Radix, `cn()`, tokens) → Storybook (CSF3 + autodocs) → QA. |

## Requirements

- **linear-pr-merge-sync** and **linear-prioritise-issues** require a [Linear MCP server](https://linear.app/docs/mcp) (`mcp__linear-server__*` tools).
- **design-component** requires a Figma MCP server (for extracting design specs from Figma), plus a React + Tailwind project using [class-variance-authority](https://cva.style/), [Radix UI](https://www.radix-ui.com/), and [Storybook](https://storybook.js.org/). It targets the latest Storybook (10.x).

## License

[MIT](LICENSE) © 2026 Nick Tarasov
