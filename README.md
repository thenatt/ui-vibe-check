# UI Vibe Check

A set of Cursor command files that systematically raise the visual and interaction quality of AI-generated UI. Apply them sequentially after any AI generates a first draft.

---

## The Three Files

| File | Job | Scope |
|---|---|---|
| `laws-of-ux.md` | Fix usability violations | Structure, interaction logic, input handling |
| `ui-polish.md` | Fix visual roughness | Typography, spacing, shadows, animation, layout |
| `motion-patterns.md` | Fix or add Motion animations | React components using `motion/react` only |

---

## How to Use in Cursor

Reference each file using `@` in your prompt:

```
@laws-of-ux.md audit this component
@ui-polish.md audit this component
@motion-patterns.md audit this component
```

Or chain them in one prompt, in order:

```
@laws-of-ux.md @ui-polish.md @motion-patterns.md audit this component and apply all relevant fixes
```

---

## Apply in This Order

**1 → 2 → 3. Always.**

```
laws-of-ux  →  ui-polish  →  motion-patterns
```

**Why order matters:**
- `laws-of-ux` may restructure components (grouping nav items, splitting forms). Do this first so the structure is stable before visual polish is applied.
- `ui-polish` applies the visual layer on top of settled structure. Doing this before structure is final means some fixes may need to be redone.
- `motion-patterns` is applied last because Motion wraps existing elements. If structure or visual changes happen after motion is added, the motion code may need to be rewritten.

---

## When to Skip a File

| Situation | Skip |
|---|---|| Static site with no React | `motion-patterns.md` |
| No forms, inputs, or navigation | `laws-of-ux.md` (mostly) |
| CSS/HTML-only project | `motion-patterns.md` |
| Component already has Motion installed and wired | Apply all three |

---

## What Each File Fixes

### `laws-of-ux.md`
- Hit areas too small on icon buttons and touch targets
- Navigation or menus with more than 7 flat items
- Raw unformatted numbers, phone numbers, card numbers in the UI
- Buttons with async handlers that give no immediate feedback
- Loading states that render null or blank instead of a skeleton
- Input validation that rejects valid formats (phone with dashes, etc.)

### `ui-polish.md`
- Heading text wrapping with orphaned last words
- Nested elements with the same border-radius (the "pinch" problem)
- Icons that pop in/out with no transition
- Text that looks heavy or blurry on macOS
- Numbers that shift layout width when they update
- Keyframe animations on hover states (should be CSS transitions)
- Multiple elements entering simultaneously with no stagger
- Exit animations as long as entrance animations
- Buttons with equal padding on text+icon sides (optical alignment)
- Cards with hard borders instead of layered box-shadows
- Images that lose definition against variable backgrounds
- Card grids with unequal heights or misaligned internal elements

### `motion-patterns.md`
- Buttons that snap between label widths on state change
- Accordions that jump to new height instead of animating
- Elements that exist in two states with no shared transition
- Pages that swap instantly with no navigation sense
- Lists that teleport to new positions on reorder
- Tab content that replaces without coordinated exit + enter
- Draggable elements that teleport back to origin on release
- Lists with no swipe-to-dismiss affordance
- Buttons with no tactile press response
- Skeletons that hard-cut to content on load
- Async actions with no optimistic UI feedback
- Numbers that update instantly with no sense of change
- Progress bars that jump to new values mechanically

---

## Audit Mode vs Generate Mode

All three files support both modes.

**Audit mode** — apply to existing generated code:
```
@ui-polish.md audit this component and fix all violations
```

**Generate mode** — apply when building something new:
```
@ui-polish.md @motion-patterns.md build a pricing card component
```

In generate mode, the AI bakes the patterns in from the start rather than fixing after. Higher quality output, fewer passes needed.

---

## Source Material

- `laws-of-ux.md` — distilled from [Laws of UX](https://www.userinterface.wiki/laws-of-ux)
- `ui-polish.md` — distilled from [Details That Make Interfaces Feel Better](https://jakub.kr/writing/details-that-make-interfaces-feel-better)
- `motion-patterns.md` — distilled from [Motion documentation](https://motion.dev/docs/react-layout-animations) and [Animating Container Bounds](https://jakub.kr/writing/animating-container-bounds)
