# Component Implementation (Steps 2-3)

## Step 2 — Plan Component

- Check `/components/ui/` + Storybook to avoid duplicates.
- Outline props + interactions; map every Figma value to a token.

## Step 3 — Implement

### Requirements

- `React.forwardRef` for all components.
- `cn()` utility for className merging (from `@/lib/utils`).
- TypeScript interfaces with JSDoc for all props.
- Comment utility groups (layout, tokens, transitions).
- Use only token-mapped values from Figma extraction — **never hardcode colors/spacing**.
- `asChild` prop via Radix `Slot` when polymorphic rendering is needed.

### Component Checklist

- [ ] File in `src/components/ui/[name].tsx`
- [ ] `React.forwardRef` wrapping root element
- [ ] `cn()` for className merging
- [ ] TypeScript interface extending HTML element props
- [ ] JSDoc header (summary, features, tokens, usage snippet)
- [ ] Props typed + documented (defaults noted)
- [ ] Semantic typography utilities (e.g., `text-base font-semibold leading-6`)
- [ ] No hard-coded colors/spacing; all from Figma variables → tokens
- [ ] CVA variants if more than one visual style
- [ ] Hover/focus states: `focus-visible:ring-2 focus-visible:ring-ring/20`
- [ ] Touch targets: minimum 44px on interactive elements
- [ ] Images use `loading="lazy"` where applicable
- [ ] Conditional rendering covers optional sub-elements
- [ ] Added to Storybook under the correct component group

### Reuse Before Building

Always check existing components before creating new UI:

| Component | Key Props | File |
|-----------|-----------|------|
| **Button** | `variant`, `size`, `asChild` | `button.tsx` |
| **Avatar** | `size`, variant (image/text) | `avatar.tsx` |
| **Input** | `type`, `disabled`, `invalid` | `input.tsx` |
| **Card** | header/body/footer slots | `card.tsx` |
| **Badge** | `variant` | `badge.tsx` |
| **Icon** | `name`, inherits `currentColor` | `icon.tsx` |
| **Tooltip** | `content`, `delay`, `offset` | `tooltip.tsx` |
| **Tabs** | active indicator | `tabs.tsx` |
| **Dialog** | Radix-backed | `dialog.tsx` |

Run `ls src/components/ui/` to see available components.

## Component Patterns

### CVA Variant System

The canonical pattern for components with multiple visual styles. Button is the reference example:

```tsx
import { cva, type VariantProps } from "class-variance-authority"
import { Slot } from "@radix-ui/react-slot"
import { cn } from "@/lib/utils"

const buttonVariants = cva(
  // Base: 10px radius, 200ms transitions
  "inline-flex items-center justify-center gap-2 whitespace-nowrap font-normal transition-all duration-200 focus-visible:outline-none focus-visible:ring-2 focus-visible:ring-offset-2 disabled:pointer-events-none disabled:opacity-50 rounded-[10px]",
  {
    variants: {
      variant: {
        primary:             "bg-primary text-primary-foreground hover:bg-primary/90",
        "secondary-neutral": "bg-background text-foreground border border-border",
        "secondary-color":   "bg-secondary text-secondary-foreground",
        "tertiary-gray":     "bg-transparent text-muted-foreground hover:bg-muted",
        link:                "text-primary underline-offset-4 hover:underline",
      },
      size: {
        sm: "h-8 px-3 py-2 [&_svg]:size-4",         // 32px
        md: "h-10 px-4 py-2.5 [&_svg]:size-5",       // 40px
        lg: "h-12 px-6 py-3 [&_svg]:size-5",          // 48px
        icon: "h-10 w-10 [&_svg]:size-5",
        "icon-sm": "h-8 w-8 [&_svg]:size-4",
      },
    },
    defaultVariants: { variant: "primary", size: "md" },
  }
)

// Props = HTML attrs + CVA variants + custom
export interface ButtonProps
  extends React.ButtonHTMLAttributes<HTMLButtonElement>,
    VariantProps<typeof buttonVariants> {
  asChild?: boolean
  iconOnly?: boolean
  tooltipLabel?: string
}
```

### Radix Slot (asChild Pattern)

Used when a component needs to render as a different element while keeping its styles:

```tsx
import { Slot } from "@radix-ui/react-slot"

const Button = React.forwardRef<HTMLButtonElement, ButtonProps>(
  ({ asChild = false, variant, size, className, ...props }, ref) => {
    const Comp = asChild ? Slot : "button"
    return <Comp ref={ref} className={cn(buttonVariants({ variant, size, className }))} {...props} />
  }
)

// Usage — renders <a> with Button styles
<Button asChild variant="primary">
  <a href="/dashboard">Go to dashboard</a>
</Button>
```

### Radix UI Primitives

Complex interactive components wrap Radix primitives with project styling. Radix handles accessibility, keyboard navigation, and state management.

| Component | Radix Package | File |
|-----------|---------------|------|
| Dialog | `@radix-ui/react-dialog` | `dialog.tsx` |
| Dropdown Menu | `@radix-ui/react-dropdown-menu` | `dropdown-menu.tsx` |
| Tooltip | `@radix-ui/react-tooltip` | `tooltip.tsx` |
| Collapsible | `@radix-ui/react-collapsible` | `collapsible.tsx` |
| Avatar | `@radix-ui/react-avatar` | `avatar.tsx` |
| Checkbox | `@radix-ui/react-checkbox` | `checkbox.tsx` |
| Popover | `@radix-ui/react-popover` | `popover.tsx` |
| Slot | `@radix-ui/react-slot` | Used in `button.tsx` |

### Icon-Only with Tooltip

Buttons with `iconOnly` auto-wrap with Tooltip when `tooltipLabel` is provided:

```tsx
<Button variant="tertiary-gray" size="sm" iconOnly tooltipLabel="Delete">
  <Icon name="trash" />
</Button>
```

This auto-adjusts size (sm→icon-sm, md→icon-md, lg→icon-lg) and wraps with `<TooltipProvider delayDuration={100}>`.

### Composition Over Configuration

Prefer composing small components over building mega-components:

```tsx
// Good — compose existing primitives
function MemberChip({ name, avatarUrl, onRemove }: MemberChipProps) {
  return (
    <Tag variant="default" size="md" removable onRemove={onRemove}>
      <Avatar src={avatarUrl} size="sm" />
      <span className="text-sm text-content-primary">{name}</span>
    </Tag>
  )
}
```
