# Theming Architecture

## How It Works

The theming system has two layers:

1. **Tailwind config** (`tailwind.config.js`) — primary token definitions used directly in classes like `bg-primary`, `text-content-secondary`
2. **CSS variables** (`src/index.css`) — shadcn compatibility layer providing theme-aware variables like `bg-background`, `text-foreground`

Dark mode uses `darkMode: ["class"]` — the `.dark` class on `<html>` swaps CSS variable values. No React context or ThemeProvider needed.

## CSS Variable Layer

The full variable setup lives in `src/index.css`. Key variables:

### Light (`:root`)

| Variable | HSL Value | Resolves To |
|----------|-----------|-------------|
| `--background` | `0 0% 100%` | white |
| `--foreground` | `20 14% 4%` | near-black |
| `--primary` | `221 83% 53%` | #2563eb (example brand color — replace) |
| `--primary-foreground` | `0 0% 100%` | white |
| `--secondary` | `0 0% 94%` | light gray |
| `--muted` | `0 0% 94%` | light gray |
| `--muted-foreground` | `0 0% 42%` | medium gray |
| `--border` | `0 0% 82%` | border gray |
| `--ring` | `221 83% 53%` | example brand color |
| `--radius` | `0.625rem` | 10px |
| `--destructive` | `0 65% 67%` | red |

### Dark (`.dark`)

| Variable | HSL Value | Change |
|----------|-----------|--------|
| `--background` | `222.2 84% 4.9%` | dark blue-black |
| `--foreground` | `210 40% 98%` | near-white |
| `--secondary` | `217.2 32.6% 17.5%` | dark gray |
| `--muted` | `217.2 32.6% 17.5%` | dark gray |
| `--border` | `217.2 32.6% 17.5%` | dark gray |

> **Note**: `--primary` stays the same in both themes (the example brand color).

### Sidebar Variables

Sidebar has its own variable set mapped via `@theme inline` in index.css. Read the file directly for the full set — they follow the same light/dark pattern.

## Using Tokens in Components

### Shadcn-Style (theme-aware via CSS variables)

```tsx
<div className="bg-background text-foreground border-border" />
<div className="bg-card text-card-foreground" />
<div className="bg-muted text-muted-foreground" />
<button className="bg-primary text-primary-foreground" />
```

### Direct Tailwind Config Tokens

```tsx
<p className="text-content-primary" />       // #0a0a0a — NOT theme-aware
<div className="bg-background-muted" />      // #f5f5f5 — NOT theme-aware
<div className="border-border-strong" />     // #d4d4d4 — NOT theme-aware
```

### When to Use Which

- **Theme-aware variables** (`bg-background`, `text-foreground`) — for structural elements that should adapt to dark mode
- **Direct tokens** (`text-content-primary`, `bg-background-muted`) — for Figma-matched values in light-mode-only contexts
- **Prefer a token over inline hex** — add a token (e.g. `bg-primary`) rather than hardcoding a hex; reserve inline hex for a one-off Figma value that has no token yet

## Dark Mode Toggle

```ts
document.documentElement.classList.add('dark')     // Enable
document.documentElement.classList.remove('dark')  // Disable
document.documentElement.classList.toggle('dark')  // Toggle
```

## Adding New Theme-Aware Tokens

When you need a new CSS variable that responds to dark mode:

```css
/* In index.css, inside @layer base */
:root {
  --my-new-token: 0 0% 95%;    /* Light value — HSL without hsl() wrapper */
}
.dark {
  --my-new-token: 220 15% 20%;  /* Dark value */
}
```

Then use it in Tailwind classes via arbitrary values: `bg-[hsl(var(--my-new-token))]`

## Accessibility

### Animation Conventions

| Context | Duration | Easing |
|---------|----------|--------|
| CSS transitions (standard) | 180ms | ease-out |
| CSS transitions (fade) | 200ms | ease-out |
| Framer Motion springs | 350ms | `{ type: "spring", bounce: 0 }` |
| Mount animations | Skip | `initial={false}` |

### Reduced Motion

```tsx
// In framer-motion components
import { useReducedMotion } from "framer-motion"

const prefersReduced = useReducedMotion()
const transition = prefersReduced
  ? { duration: 0 }
  : { type: "spring", duration: 0.35, bounce: 0 }
```

### High Contrast & Forced Colors

```css
@media (prefers-contrast: high) {
  :root { --border: 0 0% 0%; --foreground: 0 0% 0%; }
}

@media (forced-colors: active) {
  .button { border: 2px solid currentColor; }
}
```
