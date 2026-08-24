# QA — Design Engineering Review (Step 6)

Run two verification passes before shipping.

## Pass A — Visual & Functional

- Manual UI check in browser.
- Keyboard navigation works.
- Hover/focus states visible.
- Responsive at common breakpoints.
- Long and short content renders correctly.
- Run lints on touched files.
- Testing done in Storybook (stories + `play` interaction tests) or the actual app.

## Pass B — Design Engineering Standards

Design engineering standards pass:

- [ ] No layout shift on dynamic content
- [ ] Animations have `prefers-reduced-motion` support
- [ ] Touch targets >= 44px; hover effects disabled on touch (`@media (hover: hover)`)
- [ ] Keyboard nav works; icon buttons have `aria-label`
- [ ] No `transition: all`; specify exact properties
- [ ] z-index uses fixed scale or `isolation: isolate`
- [ ] Inputs >= 16px to prevent iOS zoom
- [ ] Easing follows the decision flowchart (ease-out for enter/exit, ease-in-out for movement)

**If any item fails, fix before shipping.**
