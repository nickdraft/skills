# Animation Patterns (framer-motion)

## Golden Rule

Only animate `transform`, `opacity`, `background-color`, `border-radius`, `border-color`.

**Never animate `padding`, `margin`, `height`, `width`** — they trigger layout recalculation.

## Spring Config

```tsx
import { motion } from "framer-motion"
const spring = { type: "spring", duration: 0.35, bounce: 0 }

<motion.div
  animate={hovered ? hoverState : defaultState}
  initial={false}
  transition={spring}
>
```

- `initial={false}` — no entrance animation on mount.
- `useState` + `onMouseEnter/Leave` over `whileHover` for multi-property morphs.

## Prevent Layout Shift on Hover

Don't animate padding. Instead:

1. Absolute-position the moving element.
2. Give siblings fixed spacing.
3. Animate with `translateX/Y`.

## Color Format

Use `rgba()` for both states so framer-motion interpolates correctly:

```ts
const defaultState = { backgroundColor: "rgba(247, 247, 247, 0)" }
const hoverState   = { backgroundColor: "rgba(247, 247, 247, 1)" }
```

## Animation Checklist

- [ ] Only transform/opacity/paint properties animated.
- [ ] Spring with `bounce: 0` for product UI.
- [ ] `initial={false}` to skip mount animation.
- [ ] Moving elements absolutely positioned.
- [ ] Text containers have fixed dimensions.
- [ ] `prefers-reduced-motion` respected.
