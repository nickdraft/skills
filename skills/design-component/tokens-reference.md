# Tokens Reference

> **Example token set** — adapt names and values to your own design system. The `primary` color below is a placeholder.

Quick lookup for all design tokens defined in `tailwind.config.js`.

> **Source of truth**: Always check `tailwind.config.js` directly if a value seems off — this file is a convenience reference.

## Colors

### Brand & Content

```js
primary: { DEFAULT: '#2563eb', foreground: '#ffffff' }   // example — replace with your brand color

content: { primary: '#0a0a0a', secondary: '#525252', tertiary: '#737373' }
```

- `bg-primary` / `text-primary-foreground` — brand surface and its foreground
- `text-foreground` (`#0a0a0a`), `text-muted-foreground` (`#737373`) — content layers

### Background

```js
background: { DEFAULT: '#ffffff', subtle: '#fafafa', muted: '#f5f5f5' }
```

- `bg-background` / `bg-muted` — page and muted surfaces

### Border

```js
border: { DEFAULT: '#e5e5e5', strong: '#d4d4d4' }
```

- `border-border` — default border

### Semantic

Add semantic tokens (success/warning/destructive) as your system needs.

## Typography

### Display — your display face (example: Inter / system-ui) (`font-display`)

| Token | Size | Line Height | Weight | Letter Spacing |
|-------|------|-------------|--------|----------------|
| `display-large` | 96px | 0.85 | 800 | -0.02em |
| `display-medium` | 64px | 1.0 | 800 | -0.02em |
| `display-small` | 48px | 1.1 | 800 | -0.01em |

### Title — Inter SemiBold (`font-sans`)

| Token | Size | Line Height | Weight |
|-------|------|-------------|--------|
| `title-screen` | 30px | 34px | 600 |
| `title-section` | 26px | 32px | 600 |
| `title-subsection` | 22px | 28px | 600 |
| `title-body` | 18px | 24px | 600 |
| `title-group` | 14px | 20px | 600 |

### Body — Inter (`font-sans`)

| Token | Size | Line Height | Weight |
|-------|------|-------------|--------|
| `body-large-bold` | 16px | 24px | 600 |
| `body-large` | 16px | 24px | 400 |
| `body-default-bold` | 14px | 22px | 600 |
| `body-default` | 14px | 22px | 400 |
| `body-small` | 12px | 18px | 400 |

### Font Families

```js
sans: ['Inter', 'system-ui', '-apple-system', 'BlinkMacSystemFont', 'Segoe UI', 'sans-serif']
display: ['Inter', 'system-ui', 'sans-serif']   // swap for your display face
```

> **Gotcha**: Custom `text-body-*` / `text-title-*` classes may not generate correctly. Prefer standard Tailwind:
> - `text-body-large-bold` → `text-base font-semibold leading-6`
> - `text-body-default` → `text-sm leading-[22px]`
> - `text-title-body` → `text-lg font-semibold leading-6`

## Spacing

```js
'0': '0px'    '0.5': '2px'   '1': '4px'    '1.5': '6px'
'2': '8px'    '3': '12px'    '4': '16px'   '6': '24px'   '8': '32px'
```

## Border Radius

```js
'none': '0'          'sm': '6px'          'md': '8px'          'lg': '12px'
'mobile-sm': '10px'  'mobile-md': '16px'  'mobile-lg': '24px'  'mobile-xl': '32px'
'mobile-2xl': '48px' 'full': '9999px'
```

Buttons use `rounded-[10px]` (mobile-sm). Tags/pills use `rounded-[48px]` (mobile-2xl).

## Token Usage Guidelines

1. **Direct Tailwind tokens** → `bg-primary`, `text-content-secondary`, `border-border-strong`
2. **Shadcn variable classes** → `bg-background`, `text-foreground`, `border-border` (theme-aware)
3. **Inline hex** → only for a one-off Figma value with no token yet; prefer adding a token and using `bg-primary`
4. **Never hardcode** in component files — if used more than once, add to `tailwind.config.js`

## Contrast Validation

| Combination | Ratio | WCAG |
|-------------|-------|------|
| #0a0a0a on #ffffff | 19.8:1 | AAA |
| #525252 on #ffffff | 7.8:1 | AAA |
| #737373 on #ffffff | 4.7:1 | AA |
| #ffffff on #2563eb | 5.2:1 | AA |
