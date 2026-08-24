---
name: design-component
description: "Design system skill for building UI components, looking up design tokens, understanding theming, and working with component patterns. Use this skill when: building UI components from Figma specs, extracting design tokens from Figma, adding components to Storybook, looking up color/typography/spacing token values, asking about dark mode or CSS variables, working with CVA variants or Radix primitives, adding new tokens to tailwind.config.js, or understanding the component architecture (forwardRef, cn(), asChild). This is the go-to skill for ANY design system question — whether building new components or just referencing existing tokens and patterns. Triggers on: design system, component build, UI component, design tokens, Figma to code, design to code, implement Figma design, build component from Figma, Figma handoff, Dev Mode, Figma extraction, Figma MCP, design spec, visual spec, figma_execute, figma_navigate, figma_capture_screenshot, Storybook, stories, .stories.tsx, CSF, CSF3, StoryObj, Meta, argTypes, args, autodocs, Storybook controls, play function, interaction test, MDX docs, decorators, globalTypes, component documentation, component showcase, framer-motion, spring animation, tailwind config, CSS variables, dark mode, theming, CVA variants, component patterns, cn utility, forwardRef, asChild, Radix primitives, color tokens, spacing tokens, typography scale, shadcn variables, primary/accent color."
---

# Design Component Guide

> **Requires:** a Figma MCP server for design extraction, plus a React + Tailwind project using class-variance-authority, Radix UI, and Storybook (latest, 10.x). framer-motion for animations.

Workflow: **Figma -> Build -> Storybook -> QA**. Figma is always the source of truth.

## Quick Reference

| Category                                          | When to Use                                              |
| ------------------------------------------------- | -------------------------------------------------------- |
| [Figma Extraction](figma-extraction.md)           | Pulling specs, structure, and tokens from Figma          |
| [Implementation](implementation.md)               | Building components, CVA variants, Radix patterns, cn()  |
| [Tokens Reference](tokens-reference.md)           | Looking up color, typography, spacing, radius values     |
| [Theming](theming.md)                             | CSS variables, dark mode, adding theme-aware tokens      |
| [Storybook](storybook-docs.md)                      | CSF3 stories, autodocs, sidebar hierarchy via Meta title |
| [Animation Patterns](animation-patterns.md)       | framer-motion springs, hover morphs, layout shift fixes  |
| [QA Checklist](qa-checklist.md)                    | Visual review, design engineering standards, lint        |

## Core Principles

1. **Mirror Figma exactly** — no guessing values; always pull from Figma MCP.
2. **Build composable TS components** in `src/components/ui`.
3. **Use existing tokens/utilities only** — see `tailwind.config.js` and your CSS variables file.
4. **Favor framer-motion springs** for hover/morph animations (see [Animation Patterns](animation-patterns.md)). Pure CSS only for simple color/opacity fades.
5. **Reuse existing components** — Button, Avatar, Input, Card, Badge, Icon before writing custom UI.
6. **Test in Storybook** — ask before creating standalone test files.
7. **Document with stories** — every component gets a colocated `*.stories.tsx` file (CSF3 + autodocs), grouped via its Meta `title`. See [Storybook](storybook-docs.md).

## Build Steps (Summary)

| Step | Action | Details |
| ---- | ------ | ------- |
| 1 | Extract from Figma | Use Figma MCP tools to get structure, tokens, screenshots ([details](figma-extraction.md)) |
| 2 | Plan Component | Check for duplicates, outline props, map Figma values to tokens |
| 3 | Implement | React.forwardRef, cn(), TypeScript, JSDoc, token-mapped values only ([details](implementation.md)) |
| 4 | Storybook | Write a CSF3 `*.stories.tsx` (Meta + stories, `tags: ['autodocs']`); group via Meta `title` ([details](storybook-docs.md)) |
| 5 | QA | Visual + functional pass, then design engineering standards pass ([details](qa-checklist.md)) |

## Success Criteria

- Component matches Figma precisely (verified via MCP-extracted values).
- All values trace back to Figma variables → design tokens.
- Passes both QA passes (visual + design engineering standards).
- Storybook story renders, Controls work, and autodocs educates.
- Zero lint/TS errors; accessibility + keyboard support confirmed.

## Quick Commands

```bash
ls src/components/ui/          # Inspect components
rg "ComponentName" src          # Search usages
rg -l "satisfies Meta" src        # find existing stories
npm run storybook                               # open the dev server
npx eslint src/components/ui/component-name.tsx
```

## Reference Files

For detailed guidance on specific topics:

- [figma-extraction.md](figma-extraction.md) — Figma MCP extraction workflow and scripts
- [implementation.md](implementation.md) — Component patterns (CVA, Radix, asChild), checklist, reusable inventory
- [tokens-reference.md](tokens-reference.md) — All color, typography, spacing, radius token values
- [theming.md](theming.md) — CSS variable architecture, dark mode, adding theme-aware tokens
- [storybook-docs.md](storybook-docs.md) — Storybook setup, CSF3 stories, autodocs, MDX, interaction tests
- [animation-patterns.md](animation-patterns.md) — framer-motion springs, hover morphs, layout shift prevention
- [qa-checklist.md](qa-checklist.md) — QA passes and design engineering review
