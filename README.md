# ui-vibe-check

AI is great at generating UI. It's terrible at caring about the details. This repo fixes that.

Three Cursor command files that systematically raise the quality of AI-generated UI — from "it works" to "it's good." Apply them sequentially after any first draft.

---

## The Three Files

| File | Job | Scope |
|---|---|---|
| `dont-make-them-think.md` | Fix usability violations | Structure, interaction logic, input handling |
| `pixel-therapy.md` | Fix visual roughness | Typography, spacing, shadows, animation, layout |
| `make-it-move.md` | Fix or add Motion animations | React components using `motion/react` only |

---

## How to Use in Cursor

Reference each file using `@` in your prompt:

```
@dont-make-them-think.md audit this component
@pixel-therapy.md audit this component
@make-it-move.md audit this component
```

Or run the full vibe check in one shot:

```
@dont-make-them-think.md @pixel-therapy.md @make-it-move.md audit this component and apply all relevant fixes
```

---

## Apply in This Order

**1 → 2 → 3. Always.**

```
dont-make-them-think  →  pixel-therapy  →  make-it-move
```

**Why order matters:**
- `dont-make-them-think` may restructure components — grouping nav items, splitting forms, fixing interaction logic. Structure needs to be stable before the visual layer goes on top.
- `pixel-therapy` applies the visual layer. Doing this before structure is settled means some fixes may need to be redone.
- `make-it-move` wraps existing elements with Motion. If structure or visual changes happen after, the motion code breaks. Always last.

---

## When to Skip a File

| Situation | Skip |
|---|---|
| Static site with no React | `make-it-move.md` |
| No forms, inputs, or navigation | `dont-make-them-think.md` (mostly) |
| CSS / HTML-only project | `make-it-move.md` |
| React project with Motion already wired | Apply all three |

---

## What Each File Fixes

### `dont-make-them-think.md`
- Hit areas too small on icon buttons and touch targets
- Navigation or menus with more than 7 flat items
- Raw unformatted numbers, phone numbers, card numbers in the UI
- Buttons with async handlers that give no immediate feedback
- Loading states that render null or blank instead of a skeleton
- Input validation that rejects valid formats (phone with dashes, etc.)

### `pixel-therapy.md`
- Heading text wrapping with orphaned last words
- Nested elements with the same border-radius (the "pinch" problem)
- Icons that pop in/out with no transition
- Text that looks heavy or blurry on macOS
- Numbers that shift layout width when they update
- Keyframe animations on hover states (should be CSS transitions)
- Multiple elements entering simultaneously with no stagger
- Exit animations as long as entrance animations
- Buttons with equal padding on text + icon sides (optical alignment)
- Cards with hard borders instead of layered box-shadows
- Images that lose definition against variable backgrounds
- Card grids with unequal heights or misaligned internal elements

### `make-it-move.md`
- Buttons that snap between label widths on state change
- Accordions that jump to new height instead of animating
- Elements that exist in two states with no shared transition
- Pages that swap instantly with no sense of navigation
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

## Two Modes

Both modes work with all three files.

**Audit mode** — fix an existing first draft:
```
@pixel-therapy.md audit this component and fix all violations
```

**Generate mode** — bake quality in from the start:
```
@pixel-therapy.md @make-it-move.md build a pricing card component
```

Generate mode is better when you can use it. The AI applies the patterns while building rather than retrofitting — fewer passes, higher baseline.

---

## Source Material

- `dont-make-them-think.md` — distilled from [Laws of UX](https://www.userinterface.wiki/laws-of-ux)
- `pixel-therapy.md` — distilled from [Details That Make Interfaces Feel Better](https://jakub.kr/writing/details-that-make-interfaces-feel-better)
- `make-it-move.md` — distilled from [Motion docs](https://motion.dev/docs/react-layout-animations) and [Animating Container Bounds](https://jakub.kr/writing/animating-container-bounds)
