# Storybook Integration (Step 4)

> **Version:** targets the **latest Storybook (10.x)** with Vite. Key paths used below — test utils from `storybook/test`, doc blocks from `@storybook/addon-docs/blocks`, docs via the `@storybook/addon-docs` addon — are current as of v10. Always check the [latest docs](https://storybook.js.org/docs) rather than pinning to an older major.

## Philosophy

- Every component gets a colocated `*.stories.tsx` file — the stories **are** the living documentation.
- **Autodocs** turns those stories into a docs page automatically; you rarely hand-write a page.
- Be pragmatic: one story per state that changes how a developer uses the component; expose the rest as **Controls**.
- A developer should understand a component in 60 seconds — Primary story + Controls + auto props table get them there.

## Setup

Initialize Storybook once per project:

```bash
npm create storybook@latest
```

The v10 installer asks **Recommended** (bundles docs, testing, and a11y) vs **Minimal**, or take them explicitly with `--features docs test a11y`. It scaffolds `.storybook/main.ts`, `.storybook/preview.ts`, and example stories.

`.storybook/main.ts` — where stories live, the framework, and addons:

```ts
import type { StorybookConfig } from '@storybook/react-vite';

const config: StorybookConfig = {
  stories: ['../src/**/*.mdx', '../src/**/*.stories.@(js|jsx|mjs|ts|tsx)'],
  framework: '@storybook/react-vite',
  addons: ['@storybook/addon-docs'],
};
export default config;
```

`.storybook/preview.ts` — global parameters, decorators, and (optionally) autodocs for every component:

```ts
import type { Preview } from '@storybook/react-vite';

const preview: Preview = {
  parameters: { /* layout, backgrounds, controls, ... */ },
  tags: ['autodocs'], // generate a Docs page for every component (or set it per-Meta instead)
};
export default preview;
```

Run and build:

```bash
npm run storybook        # dev server
npm run build-storybook  # static build
```

## Sidebar Hierarchy

The sidebar tree is derived from each Meta's `title`, slash-delimited — there is no hand-rolled groups registry. `title: 'Actions/Button'` files Button under an **Actions** folder. Omit `title` and Storybook auto-titles from the file path.

Keep a small, generic taxonomy so related components sit together:

| Folder | Example Components |
|---|---|
| **Actions** | Button, Menu, Tabs |
| **Inputs** | Input, Textarea, Checkbox, Radio, Switch, Select |
| **Data Display** | Card, Avatar, Table, List |
| **Feedback** | Badge, Alert, Toast, Skeleton, Tooltip |
| **Overlays** | Dialog, Popover, Sheet, Dropdown Menu |
| **Layout** | Separator, Scroll Area, Resizable Panel |

Pick the closest folder for a new component; don't invent folders casually. `name` / `storyName` overrides a single story's sidebar label. When a story's display name matches the component name, Storybook hoists it (single-story hoisting).

## Canonical Stories File (CSF3)

Put a `<component>.stories.tsx` alongside each component — flat (`src/components/ui/button.stories.tsx`) or in a per-component folder (`src/components/ui/button/button.stories.tsx`); the `../src/**/*.stories.tsx` glob matches both. The default export is the **Meta**; named UpperCamelCase exports are **stories**.

```tsx
import type { Meta, StoryObj } from '@storybook/react-vite'; // match your framework package
import { fn } from 'storybook/test';
import { Button } from '@/components/ui/button';

const meta = {
  component: Button,
  title: 'Actions/Button',
  tags: ['autodocs'],
  // component-level defaults; fn() auto-spies onClick so `play` can assert on it
  args: { children: 'Button', variant: 'primary', onClick: fn() },
  argTypes: {
    variant: {
      control: 'select',
      options: ['primary', 'secondary', 'ghost', 'link'],
      description: 'Visual style',
      table: { category: 'Appearance' }, // groups the control
    },
    size: { control: 'radio', options: ['sm', 'md', 'lg'] },
    disabled: { control: 'boolean' },
  },
} satisfies Meta<typeof Button>;

export default meta;
type Story = StoryObj<typeof meta>;

// Each state / variant is one story:
export const Primary: Story = {};
export const Secondary: Story = { args: { variant: 'secondary' } };
export const Disabled: Story = { args: { disabled: true } };

// Compose by spreading another story's args:
export const SmallSecondary: Story = { args: { ...Secondary.args, size: 'sm' } };

// Custom render when a story needs a wrapper or composition:
export const WithIcon: Story = {
  render: (args) => (
    <Button {...args}>
      <Icon name="plus" /> Add item
    </Button>
  ),
};
```

Notes:
- `satisfies Meta<typeof Button>` keeps `args` / `argTypes` type-checked while `StoryObj<typeof meta>` stays inferred.
- Import `Meta` / `StoryObj` from your **framework** package (`@storybook/react-vite`, `@storybook/nextjs-vite`, …), not a generic path.
- **Callbacks:** assign `fn()` (from `storybook/test`) to callback args — it creates an auto-spy you can assert on inside `play` and logs to the Actions panel. The older `argTypes: { onClick: { action: 'clicked' } }` still logs but produces **no spy**, so `play` can't assert against it — prefer `fn()`.

## Stories as States

The old "states table" becomes stories. Write one `StoryObj` per state that changes usage — `Disabled`, `Loading`, `Empty`, each severity of an Alert — and let **Controls** toggle everything else live. Rule of thumb: if a state doesn't change how a developer uses the component, don't give it a story; expose it as an arg.

## Args & Controls

Controls auto-generate from `args` + `argTypes`. `argTypes` are inferred from your TypeScript prop types via react-docgen; manual `argTypes` override that inference.

Common control types:

| Control | Use |
|---|---|
| `boolean` | flags (`disabled`, `loading`) |
| `number` `{ min, max, step }` / `range` | numeric props |
| `text` | strings |
| `color` `{ presetColors }` | color props |
| `radio` / `check` / `select` / `multi-select` `{ options }` | enums |
| `object` / `date` | complex props |

Useful `argTypes` fields: `description`, `table.type.summary`, `table.defaultValue.summary`, `table.category` (groups controls), `mapping` (map a control string to JSX / complex value), and `if` (show a control only when another arg is set).

## Autodocs (the docs page)

`tags: ['autodocs']` — set globally in `preview.ts` **or** per-Meta (you don't need both) — generates a Docs page containing: **Title**, **Subtitle**, **Description**, the **Primary** story, an interactive **Controls** (props) table, and the remaining **Stories**. No hand-authored page layout required.

The description and props table are driven by **JSDoc / TSDoc** on the component and its prop types:

```tsx
/** A pressable button. Use `variant` to change emphasis. */
export interface ButtonProps {
  /** Visual style. @default 'primary' */
  variant?: 'primary' | 'secondary' | 'ghost' | 'link';
  /** Disable interaction. */
  disabled?: boolean;
}
```

Disable per component with `tags: ['!autodocs']`. Reorder or replace blocks via `parameters.docs.page` using doc blocks (`Title`, `Subtitle`, `Description`, `Primary`, `Controls`, `Stories`).

## MDX Docs (prose-heavy pages)

For narrative — design context, Do / Don't, usage guidance — add a `*.mdx` file alongside the stories (already covered by the `../src/**/*.mdx` glob). Import doc blocks and attach to the **full** stories exports:

```mdx
import { Meta, Primary, Controls, Canvas } from '@storybook/addon-docs/blocks';
import * as ButtonStories from './button.stories';

<Meta of={ButtonStories} />

# Button

Primary action trigger. Prefer one primary button per view.

<Primary />
<Controls />

## Variants
<Canvas of={ButtonStories.Secondary} />

## Do / Don't
- **Do** use the `link` variant for inline navigation.
- **Don't** stack multiple primary buttons in one row.
```

`<Meta of={ButtonStories} />` must reference the full set of story exports (not the component). For a standalone page not tied to a component, use `<Meta title="Guides/Getting Started" />` and omit `of`.

## Interaction Tests (play)

Replace "test in Storybook" with the `play` function — it runs after the story renders and reports in the Interactions panel:

```tsx
import { expect, fn } from 'storybook/test';

export const SubmitsForm: Story = {
  args: { onSubmit: fn() },
  play: async ({ args, canvas, userEvent }) => {
    await userEvent.type(canvas.getByLabelText('Email'), 'a@b.com');
    await userEvent.click(canvas.getByRole('button', { name: 'Submit' }));
    await expect(args.onSubmit).toHaveBeenCalled();
    await expect(canvas.getByText('Thanks!')).toBeInTheDocument();
  },
};
```

Always `await` both `userEvent` and `expect` calls. Import `expect`, `fn`, `userEvent`, `within`, etc. from `storybook/test`. Assign `fn()` to any callback arg you want to assert on (`args.onSubmit`).

Run these in CI with the **Vitest addon** (`@storybook/addon-vitest`, current default) or the legacy **`@storybook/test-runner`** — both execute every story's `play` headlessly.

## Recommended Addons

For a design-system Storybook, add:

- **`@storybook/addon-a11y`** — runs axe accessibility checks on every story; near-standard for component libraries (included in the v10 **Recommended** install).
- **Theme / dark-mode toggle** — expose a toolbar switch via `globalTypes` + a decorator in `.storybook/preview.ts`, so stories render in both themes without duplicating them:

```ts
// .storybook/preview.ts
export const globalTypes = {
  theme: {
    toolbar: { items: ['light', 'dark'], dynamicTitle: true },
  },
};
export const initialGlobals = { theme: 'light' };

const preview: Preview = {
  decorators: [
    (Story, { globals }) => (
      <div className={globals.theme === 'dark' ? 'dark' : ''}>
        <Story />
      </div>
    ),
  ],
};
```

## Optional: Status Tags

Custom tags can mark maturity — `tags: ['stable']`, `tags: ['wip']`, `tags: ['deprecated']` — and can filter the sidebar or drive a badge addon. This is optional; don't gate a component's story on having a status.

## Conventions

- One `*.stories.tsx` per component, colocated with it (flat or in a per-component folder) under `src/components/ui/`.
- Group via the Meta `title` using the shared taxonomy above.
- Enable `autodocs`; write prop docs as JSDoc so the auto props table stays accurate.
- Use realistic content in `args`, not "Lorem ipsum."
- Import `Meta` / `StoryObj` from the framework package; doc blocks from `@storybook/addon-docs/blocks`; test utils from `storybook/test`.
- Don't create standalone test HTML files — use stories + `play`.

