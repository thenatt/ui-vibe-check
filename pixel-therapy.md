---
description: Apply UI polish patterns to the current component or screen. Audits for and fixes visual roughness — text rendering, border radii, shadows, animation quality, number formatting, and optical alignment.
globs: ["**/*.tsx", "**/*.jsx", "**/*.css", "**/*.html"]
---

# UI Polish Patterns

Reference command distilled from [Details That Make Interfaces Feel Better](https://jakub.kr/writing/details-that-make-interfaces-feel-better).

When invoked, scan the current component or file for violations of the patterns below and fix them. For each fix, call out what was wrong and why it matters.

---

## Quick Reference

| Pattern | Trigger Condition | Fix |
|---|---|---|
| Text wrap balance | Headings/titles wrapping unevenly | `text-wrap: balance` |
| Concentric radius | Nested elements with same border-radius | `inner = outer - padding` |
| Icon animation | Icons popping in/out on interaction | opacity + scale + blur transition |
| Font smoothing | Text looks heavy or blurry | `-webkit-font-smoothing: antialiased` |
| Tabular nums | Numbers shifting width on update | `font-variant-numeric: tabular-nums` |
| Interruptible animations | Keyframes on hover/toggle states | Replace with CSS transitions |
| Staggered entrance | Elements appearing simultaneously | Stagger with `animation-delay` |
| Subtle exits | Exit = mirror of entrance | Shorter, less movement than entrance |
| Optical alignment | Icon+text visually off-center | Reduce padding on icon side |
| Shadows not borders | Hard borders on cards/surfaces | Multi-layer `box-shadow` with transparency |
| Outline on images | Images floating without definition | 1px outline at 10% opacity, inset |
| Grid card alignment | Cards in same row with ragged heights/misaligned CTAs | CSS Grid stretch + subgrid for internal anchors |

---

## 1. Text Wrapping — `text-wrap: balance`

Single-word orphans on the last line of a heading are a passive signal of an unpolished interface. The browser can fix this in one line.

**Rule:** Always apply `text-wrap: balance` to headings and short multi-line text. Never let the layout engine decide arbitrarily.

```css
h1, h2, h3, .card-title, .hero-text {
  text-wrap: balance;
}
```

Tailwind: `text-balance`

```tsx
// Apply at the component level, not just globally
<h2 className="text-balance text-2xl font-semibold">{title}</h2>
```

**When to apply:**
- Page headings and section titles
- Card titles that may wrap at narrow widths
- Hero copy and marketing text
- Empty state headings

**Anti-pattern:** Using `max-width` hacks (`max-width: 20ch`) to control wrapping — fragile and doesn't adapt to content length.

**Note:** Use `text-wrap: pretty` for body text — it uses a slower algorithm optimized for paragraph readability, not just the last line.

---

## 2. Concentric Border Radius

When a rounded parent contains a rounded child, the inner radius must be smaller than the outer by exactly the padding between them. Same radius on both creates a visual "pinch" — the curves fight each other.

**Rule:** `inner-radius = outer-radius - padding`. Always. No exceptions.

```
outer-radius: 20px
padding: 8px
inner-radius: 12px  ← outer - padding
```

```css
.card {
  border-radius: 20px;
  padding: 8px;
}

.card-inner {
  border-radius: 12px; /* 20 - 8 = 12 */
}
```

```tsx
// Token-based approach for consistency
const Card = ({ children }) => (
  <div className="rounded-[20px] p-2"> {/* p-2 = 8px */}
    <div className="rounded-[12px] bg-muted p-4">
      {children}
    </div>
  </div>
);
```

**When to apply:**
- Cards with highlighted inner content areas
- Buttons with background pill + icon background
- Input fields with inner icon containers
- Nested panels or inset sections

**Anti-pattern:** `rounded-lg` on both parent and child without adjusting — the corners will visually clash regardless of how subtle the radius is.

---

## 3. Animate Icons Contextually

Icons that appear or disappear in response to interaction should never just pop in/out. The lack of transition signals low craft. A 150ms fade with slight scale is the minimum bar.

**Rule:** Any conditionally rendered icon needs opacity + scale + blur together. Never opacity alone.

```tsx
// Using Motion (Framer Motion) — preferred for React
import { AnimatePresence, motion } from 'motion/react';

<AnimatePresence>
  {isHovered && (
    <motion.div
      initial={{ opacity: 0, scale: 0.8, filter: 'blur(4px)' }}
      animate={{ opacity: 1, scale: 1, filter: 'blur(0px)' }}
      exit={{ opacity: 0, scale: 0.8, filter: 'blur(4px)' }}
      transition={{ type: 'spring', duration: 0.2, bounce: 0 }}
    >
      <EditIcon />
    </motion.div>
  )}
</AnimatePresence>
```

```css
/* CSS-only fallback */
.icon-contextual {
  opacity: 0;
  transform: scale(0.85);
  filter: blur(3px);
  transition:
    opacity 150ms ease,
    transform 150ms ease,
    filter 150ms ease;
}

.parent:hover .icon-contextual {
  opacity: 1;
  transform: scale(1);
  filter: blur(0);
}
```

**When to apply:**
- Row-level action icons (edit, delete, drag handle) on hover
- Status indicators appearing after async actions
- Contextual controls that reveal on focus/hover
- Badge counts appearing on state change

**Anti-pattern:** `display: none` → `display: block` on hover. Even `opacity: 0` → `opacity: 1` alone feels flat — always pair with scale and blur.

---

## 4. Crisp Text Rendering — `-webkit-font-smoothing: antialiased`

macOS defaults to subpixel antialiasing — it renders text heavier to take advantage of LCD subpixels. On modern retina displays, this makes text look thick and slightly blurry. Grayscale antialiasing is crisper and closer to design intent.

**Rule:** Apply `antialiased` globally to the root element. This is now standard for all modern web apps.

```css
body {
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
}
```

Tailwind: add `antialiased` to the root `<html>` or `<body>` class, or in the base layer.

```tsx
// In a Next.js root layout
<html className="antialiased">
```

**When to apply:**
- Every project, globally, unconditionally.

**Caveat:** This applies to macOS/iOS only. Windows uses its own ClearType rendering and ignores this property. Test on both if font weight is critical to your design.

**Anti-pattern:** Applying `antialiased` only to specific components — inconsistent rendering within the same page looks unintentional.

---

## 5. Tabular Numbers — `font-variant-numeric: tabular-nums`

By default, digits in most fonts have variable widths (proportional). When a number updates — a counter ticking, a price changing, a timer running — the surrounding layout shifts. Tabular nums give every digit identical width, eliminating layout jitter.

**Rule:** Any number that can change value at runtime must use `tabular-nums`.

```css
.counter,
.price,
.timer,
.stat {
  font-variant-numeric: tabular-nums;
}
```

Tailwind: `tabular-nums`

```tsx
// Apply at the element level
<span className="tabular-nums text-2xl font-semibold">{count}</span>

// Or in a shared utility
const Stat = ({ value }: { value: number }) => (
  <span className="tabular-nums">{value.toLocaleString()}</span>
);
```

**When to apply:**
- Live counters and real-time metrics
- Countdown timers
- Prices with dynamic values
- Table columns with numeric data
- Progress percentages

**Caveat:** Inter (and some other fonts) visually changes numeral style when `tabular-nums` is active — numbers may appear slightly different weight or style. Always do a visual check after applying.

**Anti-pattern:** Using a monospace font just to get fixed-width digits — `tabular-nums` achieves the same result without sacrificing typographic quality.

---

## 6. Interruptible Animations

CSS transitions and keyframe animations behave differently when interrupted mid-flight. Keyframes play a fixed timeline from start to finish — if the user reverses the trigger before it completes (hover off before hover animation finishes), the element snaps or jumps. Transitions always interpolate toward the current target state, making interruptions seamless.

**Rule:** Use CSS transitions for all user-triggered state changes. Reserve keyframes for animations that play once and aren't reversible.

| Type | Behavior when interrupted | Use for |
|------|--------------------------|---------|
| CSS `transition` | Smoothly retargets toward new state | Hover, toggle, expand, focus |
| `@keyframes` | Jumps or snaps — fixed timeline | Page load entrance, skeleton pulse, one-shot notifications |

```css
/* Correct — transition for interactive state */
.btn {
  background: var(--color-primary);
  transform: scale(1);
  transition: background 150ms ease, transform 150ms ease;
}

.btn:hover {
  background: var(--color-primary-dark);
  transform: scale(1.02);
}

/* Correct — keyframe for one-shot entrance */
@keyframes slideIn {
  from { opacity: 0; transform: translateY(8px); }
}

.toast {
  animation: slideIn 300ms ease both;
}
```

**When to apply:**
- Any element with hover, focus, or active states → use transitions
- Loading states, skeleton screens → use keyframes
- Page-level entrance animations → use keyframes

**Anti-pattern:** `@keyframes` on `:hover` — the animation replays from the start on every hover, and snaps on mouse-out.

---

## 7. Split & Stagger Entering Elements

When multiple elements appear at once, they compete for attention and the arrival feels cheap. Staggering entrance animations creates a sense of hierarchy and gives each element a moment of focus.

**Rule:** Never animate sibling elements with identical timing. Add staggered delays — sections at 100ms apart, words at 80ms apart.

```css
@keyframes enter {
  from {
    transform: translateY(8px);
    filter: blur(5px);
    opacity: 0;
  }
  to {
    transform: translateY(0);
    filter: blur(0);
    opacity: 1;
  }
}

.stagger-item {
  animation: enter 800ms cubic-bezier(0.25, 0.46, 0.45, 0.94) both;
  animation-delay: calc(var(--index, 0) * 80ms);
}
```

```tsx
// Assign stagger index via inline style
{items.map((item, i) => (
  <div
    key={item.id}
    className="stagger-item"
    style={{ '--index': i } as React.CSSProperties}
  >
    {item.content}
  </div>
))}
```

**Stagger timing:**
- List items / cards: 60–80ms between items
- Page sections: 100–120ms between sections
- Words in a title: 40–60ms between words (keep it tight)

**Always combine:** opacity + blur + translateY together. Any one of them alone feels thin.

**When to apply:**
- Dashboard cards appearing on page load
- Search results populating
- Menu items entering on open
- Word-by-word title reveals

**Stagger cap — hard rule:**
- Count the sibling elements being staggered
- If count ≤ 6: stagger all of them
- If count > 6: stagger only the first 5, apply no `animation-delay` to the rest (`--index: 0` for all remaining items)
- Never let the last visible stagger delay exceed ~400ms total

**Anti-pattern:** Staggering more than 5–6 items without capping — by item 7 the delay is 480ms+ and the last item feels abandoned.

---

## 8. Subtle Exit Animations

Entrances earn attention. Exits don't need it — they're just the element leaving. An exit that mirrors the entrance in scale and duration competes with incoming content. Keep exits shorter, less movement, same blur.

**Rule:** Exit animations should be 40–50% shorter than entrances and use minimal translation.

| Property | Entrance | Exit |
|----------|----------|------|
| `translateY` | `8px → 0` | `0 → -12px` |
| `opacity` | `0 → 1` | `1 → 0` |
| `blur` | `5px → 0` | `0 → 4px` |
| Duration | `800ms` | `400–450ms` |
| Easing | `cubic-bezier(0.25, 0.46, 0.45, 0.94)` | Spring, `bounce: 0` |

```tsx
// Motion — asymmetric enter/exit
<motion.div
  initial={{ opacity: 0, y: 8, filter: 'blur(5px)' }}
  animate={{ opacity: 1, y: 0, filter: 'blur(0px)' }}
  exit={{ opacity: 0, y: -12, filter: 'blur(4px)' }}
  transition={{
    enter: { duration: 0.8, ease: [0.25, 0.46, 0.45, 0.94] },
    exit: { type: 'spring', duration: 0.45, bounce: 0 }
  }}
>
  {children}
</motion.div>
```

**When to apply:**
- Modal/drawer dismiss
- Toast notifications disappearing
- Tab content transitioning out
- List items being removed

**Anti-pattern:** Matching exit duration to entrance — the interface feels sluggish because something leaving is still demanding attention while new content is trying to arrive.

---

## 9. Optical Alignment (Not Geometric)

Math-perfect centering is often visually wrong. Shapes with visual weight on one side (arrows, chevrons, icons with asymmetric mass) need manual adjustment to *look* centered — not *be* centered.

**Rule:** Trust your eye over the numbers. Optical balance overrides geometric balance.

**Audit trigger — these are the detectable code conditions:**
- Any `<button>` containing both a text label and a directional icon (`ChevronRight`, `ArrowRight`, `→`, `ChevronDown`, `ExternalLink`, or similar) with equal left and right padding → reduce right padding by 25% (e.g. `pl-4 pr-3`)
- Any play/pause icon rendered inside a circle or square container → apply `translateX(1px)` to nudge right
- Any icon-only button using a triangle or arrow shape with no transform applied → apply `translateX(1px)` or `translateY(1px)` depending on orientation

```css
/* Icon-right button — reduce right padding to compensate for arrow visual weight */
.btn-with-arrow {
  padding-left: 16px;
  padding-right: 12px; /* not 16px */
}

/* Play button in a circle — nudge right to offset the triangle's visual left-lean */
.play-btn svg {
  transform: translateX(1px);
}
```

```tsx
// Common pattern: icon + label button
<button className="flex items-center gap-2 pl-4 pr-3 py-2">
  {label}
  <ChevronRight className="w-4 h-4" /> {/* PR intentionally less than PL */}
</button>
```

**When to apply:**
- Any button that combines text + directional icon
- Play/pause buttons in circular containers
- Arrow indicators and chevrons
- Icons with asymmetric visual weight (triangles, pointing shapes)

**Best practice:** Fix optical alignment in the SVG itself when possible — edit the viewBox to redistribute whitespace so the icon centers correctly in any container.

**Anti-pattern:** Pixel-perfect alignment via CSS flexbox/grid without checking the visual result — `align-items: center` is geometric, not optical.

---

## 10. Shadows Instead of Borders

A solid 1px border creates a flat, hard edge. Multi-layer box shadows with low transparency add depth while adapting to any background color — they work on white, colored, and image surfaces without needing a separate dark mode override.

**Rule:** Use multi-layer `box-shadow` for card and surface elevation. Reserve solid borders for structural UI (tables, form inputs, dividers).

```css
/* Base card */
.card {
  box-shadow:
    0px 0px 0px 1px rgba(0, 0, 0, 0.06),
    0px 1px 2px -1px rgba(0, 0, 0, 0.06),
    0px 2px 4px 0px rgba(0, 0, 0, 0.04);
  transition: box-shadow 150ms ease;
}

/* Hover state — subtle lift */
.card:hover {
  box-shadow:
    0px 0px 0px 1px rgba(0, 0, 0, 0.08),
    0px 2px 4px -1px rgba(0, 0, 0, 0.08),
    0px 4px 8px 0px rgba(0, 0, 0, 0.06);
}

/* Dark mode — flip to white shadows */
.dark .card {
  box-shadow:
    0px 0px 0px 1px rgba(255, 255, 255, 0.08),
    0px 1px 2px -1px rgba(255, 255, 255, 0.04),
    0px 2px 4px 0px rgba(255, 255, 255, 0.03);
}
```

**Layer breakdown:**
- Layer 1 (`0px 0px 0px 1px`): replaces the border — defines the edge
- Layer 2 (`1px 2px -1px`): close shadow — adds bottom weight
- Layer 3 (`2px 4px 0px`): ambient shadow — suggests elevation

**When to apply:**
- Cards and panels
- Dropdowns and popovers
- Modals and sheets
- Any floating surface

**Anti-pattern:** `border: 1px solid #e5e7eb` on cards — looks flat on non-white backgrounds and requires a separate dark mode value. Box-shadow transparency handles both.

---

## 11. Outline on Images

Images on variable backgrounds (colored cards, gradients, dark surfaces) often lose definition at the edge — the image blends into whatever is behind it. A 1px inset outline at low opacity gives every image a consistent edge without looking like a border.

**Rule:** Apply a 1px semi-transparent outline (inset) to all images that sit on variable backgrounds.

```css
/* Using outline (doesn't affect layout) */
.img-contained {
  outline: 1px solid rgba(0, 0, 0, 0.1);
  outline-offset: -1px; /* inset — keeps it inside the image bounds */
}

/* Dark mode */
.dark .img-contained {
  outline-color: rgba(255, 255, 255, 0.1);
}
```

```tsx
// Tailwind — utility class approach
<img
  src={src}
  className="rounded-lg outline outline-1 outline-black/10 -outline-offset-1"
/>
```

**When to apply:**
- User avatars and profile images
- Product thumbnails
- Media cards with background-image fill
- Any image where the edge color may be close to the background

**Anti-pattern:** Adding a solid border to images — it adds visual bulk and looks like an artifact from an older era of web design. The inset outline is invisible at a glance; it only activates when you need it.

---

## 12. Grid Card Alignment — Equal Heights + Anchored Internals

When AI generates card grids, two problems almost always appear together: cards in the same row have different heights (because content length varies), and internal elements like CTAs or price blocks sit at different vertical positions across cards. The result looks like a rough draft even when the content is good.

**Audit trigger — structural check, no judgment needed:**
1. Find all card grids: elements where multiple sibling components share the same parent grid or flex container
2. Check if any sibling card has a prop or className containing `featured`, `highlighted`, `popular`, `recommended`, `isActive`, or `isPrimary`
   - **If yes:** leave all card heights alone — height difference is intentional hierarchy
   - **If no:** treat all height differences as broken — apply the fix below
3. This check is binary. Do not infer intent from content or visual position.

**Rule:** Use CSS Grid `stretch` to equalize card heights, then use `subgrid` to lock internal elements to consistent row anchors across all cards.

### Step 1 — Equal heights (the baseline fix)

```css
/* Parent grid — stretch is the default, but make it explicit */
.card-grid {
  display: grid;
  grid-template-columns: repeat(auto-fit, minmax(280px, 1fr));
  align-items: stretch; /* all cards in a row grow to match the tallest */
  gap: 24px;
}

/* Each card fills its grid cell */
.card {
  height: 100%;
  display: flex;
  flex-direction: column;
}
```

```tsx
<div className="grid grid-cols-3 gap-6 items-stretch">
  {plans.map(plan => (
    <PricingCard key={plan.id} {...plan} />
  ))}
</div>
```

This alone fixes ragged bottom edges. Cards in the same row now share the same height. But the CTA button will still float at different heights if content above it varies.

### Step 2 — Anchored internals with subgrid (the complete fix)

Subgrid lets child elements inside each card participate in the parent grid's row tracks. This means the price block, CTA, and feature list all snap to the same horizontal line across all cards — regardless of how much content is above them.

```css
.card-grid {
  display: grid;
  grid-template-columns: repeat(3, 1fr);
  /* Define row tracks for each internal zone */
  grid-template-rows: auto; /* rows size to tallest content in that zone */
  gap: 24px;
}

.card {
  display: grid;
  grid-row: span 4; /* how many row tracks this card spans */
  grid-template-rows: subgrid; /* inherit row tracks from parent */
}

/* Now each zone inside every card snaps to the same row */
.card-header   { grid-row: 1; } /* plan name + description */
.card-price    { grid-row: 2; } /* price block */
.card-cta      { grid-row: 3; } /* CTA button */
.card-features { grid-row: 4; } /* feature list */
```

```tsx
// React + Tailwind (Tailwind 3.3+ supports subgrid)
const PricingCard = ({ name, description, price, cta, features }) => (
  <div className="grid grid-rows-subgrid row-span-4 rounded-2xl bg-card p-6 gap-4">
    <div> {/* row 1: header */}
      <h3 className="text-lg font-semibold">{name}</h3>
      <p className="text-muted-foreground text-sm">{description}</p>
    </div>
    <div className="tabular-nums text-4xl font-bold"> {/* row 2: price */}
      {price}
    </div>
    <button className="w-full">{cta}</button> {/* row 3: CTA */}
    <ul className="space-y-2 text-sm"> {/* row 4: features */}
      {features.map(f => <li key={f}>✓ {f}</li>)}
    </ul>
  </div>
);

// Parent grid
<div className="grid grid-cols-3 grid-rows-[auto_auto_auto_auto] gap-6">
  {plans.map(plan => <PricingCard key={plan.id} {...plan} />)}
</div>
```

**Fallback if subgrid isn't viable** (older codebase, Tailwind < 3.3, or you need IE11 compatibility):

```css
/* Push the CTA to the bottom of its card using flex */
.card {
  display: flex;
  flex-direction: column;
}

.card-features {
  flex: 1; /* grows to fill available space, pushing CTA to bottom */
}

.card-cta {
  margin-top: auto; /* always sticks to the bottom of the card */
}
```

This is weaker than subgrid — it only aligns the CTA at the bottom, not the price block or other mid-card elements — but it solves 80% of the problem with zero browser risk.

**When to apply:**
- Pricing plan cards
- Feature comparison grids
- Testimonial or quote cards with variable length quotes
- Product cards with titles, descriptions, prices, and CTAs
- Any `grid` or `flex-wrap` layout where sibling cards have variable content height

**Anti-pattern:** Hardcoding a fixed `height` or `min-height` on cards — it works until content changes, then cards either clip or leave awkward empty space. Always let the grid drive height, not the card itself.