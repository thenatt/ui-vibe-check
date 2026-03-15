---
description: Apply Motion library patterns to the current component. All patterns in this file use Motion (motion/react) exclusively — CSS-only animation patterns live in ui-polish.md. In audit mode, scan for janky or missing transitions and apply the relevant pattern. In generate mode, reach for these patterns when building interactive or animated components. IMPORTANT — audit mode rule: only apply a pattern if a clear trigger condition exists in the current code. Do not introduce patterns that have no existing counterpart to fix. If the component has no drag interaction, do not add drag. If it has no list, do not add swipe-to-dismiss. Match patterns to what exists, never invent new interactions.
globs: ["**/*.tsx", "**/*.jsx"]
---

# Motion Patterns

Concrete Motion library recipes for common UI animation problems. Each pattern solves a specific class of jank or missing polish.

**Scope:** This file is Motion (motion/react) only. For CSS transitions, stagger, entrance/exit animations, and easing — see `ui-polish.md`.

---

## Quick Reference

| Pattern | Problem it solves |
|---|---|
| Animated container width | Button/label snaps between widths on content change |
| Animated container height | Accordion/expandable section jumps to new height |
| Shared layout transition | Element moves between positions or parents with no continuity |
| Page / route transition | Pages swap instantly with no sense of navigation |
| List reorder animation | Items jump to new positions when list order changes |
| Mode switching | Tab/toggle content swaps without coordinated exit + enter |
| Drag with snap-back | Draggable element teleports back to origin on release |
| Swipe to dismiss | No gesture affordance for removing list items or cards |
| Press feedback | Buttons feel flat — no tactile response on tap/click |
| Skeleton → content | Content replaces skeleton with a jump, no handoff |
| Optimistic UI | State updates feel delayed — UI waits for server confirmation |
| Number counter | Numbers update instantly with no sense of change |
| Progress bar | Progress bar jumps to new value mechanically |

---

## Audit Mode — Pattern Matching Rules

Before applying any pattern, check for its trigger condition in the existing code:

| Pattern | Required trigger to apply |
|---|---|
| Animated container width | Element whose text/label content changes dynamically |
| Animated container height | Element that conditionally shows/hides content |
| Shared layout transition | Same logical element rendered in 2+ visual states |
| Page / route transition | Router or multi-view navigation exists |
| List reorder animation | List whose order changes programmatically |
| Mode switching | Tab, toggle, or view-switcher component exists |
| Drag with snap-back | Draggable interaction already present or explicitly designed |
| Swipe to dismiss | List of removable items exists |
| Press feedback | Clickable buttons exist — apply to all |
| Skeleton → content | Component has async loading state |
| Optimistic UI | Component has mutation with async handler |
| Number counter | Numeric value that updates at runtime |
| Progress bar | Progress or completion value that updates incrementally |

If the trigger condition is not present, skip the pattern entirely.

---

## Setup

All patterns assume Motion is installed:

```bash
npm install motion
```

```tsx
// Import from motion/react for React projects
import { motion, AnimatePresence, MotionConfig, useMotionValue, useSpring, useTransform } from 'motion/react';
```

---

## 1. Animated Container Width

Buttons that change their label — loading states, confirmation messages, multi-step flows — snap between widths by default. The container doesn't know how to interpolate between `auto` widths. Fix: measure the inner content, animate the outer container to match.

**The pattern:** outer div animates, inner div measures. Never put both on the same element or you create a resize loop.

```tsx
import { motion, MotionConfig } from 'motion/react';
import { useCallback, useEffect, useState } from 'react';

function useMeasure<T extends HTMLElement = HTMLElement>() {
  const [element, setElement] = useState<T | null>(null);
  const [bounds, setBounds] = useState({ width: 0, height: 0 });

  const ref = useCallback((node: T | null) => setElement(node), []);

  useEffect(() => {
    if (!element) return;
    const observer = new ResizeObserver(([entry]) => {
      setBounds({
        width: entry.contentRect.width,
        height: entry.contentRect.height,
      });
    });
    observer.observe(element);
    return () => observer.disconnect();
  }, [element]);

  return [ref, bounds] as const;
}

// Usage — button that animates between label states
const labels = ['Save changes', 'Saving...', 'Saved ✓'];

export function AnimatedButton() {
  const [index, setIndex] = useState(0);
  const [ref, bounds] = useMeasure();

  return (
    <MotionConfig transition={{ duration: 0.4, ease: [0.19, 1, 0.22, 1] }}>
      <motion.button
        animate={{ width: bounds.width > 0 ? bounds.width : 'auto' }}
        onClick={() => setIndex(i => (i + 1) % labels.length)}
      >
        <div ref={ref} style={{ whiteSpace: 'nowrap', padding: '8px 16px' }}>
          <motion.span
            key={labels[index]}
            initial={{ opacity: 0, filter: 'blur(6px)', scale: 0.95 }}
            animate={{ opacity: 1, filter: 'blur(0px)', scale: 1 }}
            exit={{ opacity: 0, filter: 'blur(6px)', scale: 0.95 }}
          >
            {labels[index]}
          </motion.span>
        </div>
      </motion.button>
    </MotionConfig>
  );
}
```

**Gotcha:** `bounds.width > 0 ? bounds.width : 'auto'` prevents an animation from 0 on initial render. Always guard this.

**When to apply:**
- Button label changes (loading, success, error states)
- Badge or tag content updating
- Any inline element whose text content changes dynamically

---

## 2. Animated Container Height

Accordions, expandable sections, "Read more" flows, FAQ items. Without this pattern, the container jumps to its new height instantly. With it, the height smoothly follows the content.

Same two-div principle as width: outer animates, inner measures.

```tsx
import { AnimatePresence, MotionConfig, motion } from 'motion/react';
import { useState } from 'react';

// Reuse useMeasure from Pattern 1

export function Accordion({ title, children }) {
  const [expanded, setExpanded] = useState(false);
  const [ref, bounds] = useMeasure();

  return (
    <MotionConfig transition={{ duration: 0.4, ease: [0.19, 1, 0.22, 1] }}>
      <div>
        <button onClick={() => setExpanded(e => !e)}>
          {title}
          <motion.span animate={{ rotate: expanded ? 180 : 0 }}>↓</motion.span>
        </button>

        <motion.div
          animate={{ height: bounds.height > 0 ? bounds.height : 'auto' }}
          style={{ overflow: 'hidden' }}
        >
          <div ref={ref}>
            <AnimatePresence mode="popLayout">
              {expanded && (
                <motion.div
                  initial={{ opacity: 0, filter: 'blur(4px)' }}
                  animate={{ opacity: 1, filter: 'blur(0px)' }}
                  exit={{ opacity: 0, filter: 'blur(4px)' }}
                  transition={{ duration: 0.3 }}
                >
                  {children}
                </motion.div>
              )}
            </AnimatePresence>
          </div>
        </motion.div>
      </div>
    </MotionConfig>
  );
}
```

**Gotcha:** The outer `motion.div` needs `overflow: hidden` — otherwise content is visible outside the animated bounds during the transition.

**When to apply:**
- FAQ / accordion components
- "Read more" / "Show less" content reveals
- Settings panels that expand on selection
- Any section where content conditionally appears/disappears

---

## 3. Shared Layout Transition

**Mental model:** You are not animating one element between two positions. You are rendering entirely separate components for each state — each with the same `layoutId` — and Motion animates the transition between them. The card component and the modal component are different. `layoutId` is the contract that says they are the same object visually.

This means you can change size, position, border-radius, and even DOM location between states. Motion handles the interpolation using the FLIP technique (First, Last, Inverse, Play) — it records the start position, records the end position, and plays the delta as an animation.

**Critical rule — `layoutId` elements must live outside `AnimatePresence`.**
If a `layoutId` element is inside `AnimatePresence`, the `initial` and `exit` animations fire simultaneously with the layout transition. They fight each other — the element tries to morph position AND fade at the same time. The result is broken, especially when `opacity` is involved.

Structure: `layoutId` element outside, `AnimatePresence` wraps only the content inside the expanded state.

```tsx
// ✗ Wrong — layoutId inside AnimatePresence
<AnimatePresence>
  {selected && (
    <motion.div layoutId="card"> {/* exit animation fights layout transition */}
      <Content />
    </motion.div>
  )}
</AnimatePresence>

// ✓ Correct — layoutId outside, AnimatePresence wraps only inner content
<motion.div layoutId="card"> {/* morphs freely, no interference */}
  <AnimatePresence mode="popLayout">
    {selected && (
      <motion.div
        key={selected}
        initial={{ opacity: 0, filter: 'blur(4px)', scale: 0.95, y: 32 }}
        animate={{ opacity: 1, filter: 'blur(0px)', scale: 1, y: 0 }}
        exit={{   opacity: 0, filter: 'blur(4px)', scale: 0.95, y: 32 }}
        transition={{ type: 'spring', duration: 0.55, bounce: 0 }}
      >
        <Content />
      </motion.div>
    )}
  </AnimatePresence>
</motion.div>
```

**Example — card grid expanding into a modal (most common real-world use case):**

```tsx
import { AnimatePresence, motion } from 'motion/react';
import { useState } from 'react';

const items = [
  { id: 'a', title: 'First',  color: '#4f46e5', description: 'Detail content for first item.' },
  { id: 'b', title: 'Second', color: '#0891b2', description: 'Detail content for second item.' },
  { id: 'c', title: 'Third',  color: '#059669', description: 'Detail content for third item.' },
];

export function CardGrid() {
  const [selected, setSelected] = useState<string | null>(null);
  const selectedItem = items.find(i => i.id === selected);

  return (
    <>
      {/* Grid — each card has a layoutId */}
      <div style={{ display: 'flex', gap: 12 }}>
        {items.map(item => (
          <motion.div
            key={item.id}
            layoutId={item.id}             // ← contract: this = the expanded modal
            onClick={() => setSelected(item.id)}
            transition={{ type: 'spring', duration: 0.55, bounce: 0.1 }}
            style={{
              width: 80, height: 80,
              borderRadius: 12,
              background: item.color,
              cursor: 'pointer',
            }}
          />
        ))}
      </div>

      {/* Expanded modal */}
      <AnimatePresence>
        {selected && selectedItem && (
          <>
            {/* Backdrop — inside AnimatePresence, no layoutId */}
            <motion.div
              initial={{ opacity: 0 }}
              animate={{ opacity: 0.5 }}
              exit={{ opacity: 0 }}
              onClick={() => setSelected(null)}
              style={{
                position: 'fixed', inset: 0,
                background: '#000',
                zIndex: 10,
              }}
            />

            {/* Expanded card — same layoutId as the thumbnail, outside AnimatePresence */}
            <motion.div
              layoutId={selected}          // ← same id = Motion morphs from thumbnail
              transition={{ type: 'spring', duration: 0.55, bounce: 0.1 }}
              style={{
                position: 'fixed',
                top: '50%', left: '50%',
                x: '-50%', y: '-50%',
                width: 320, height: 400,
                borderRadius: 24,
                background: selectedItem.color,
                zIndex: 20,
                padding: 24,
              }}
            >
              {/* Inner content uses AnimatePresence — separate from the morphing container */}
              <AnimatePresence mode="popLayout">
                <motion.div
                  key={selected}
                  initial={{ opacity: 0, filter: 'blur(4px)', scale: 0.95, y: 16 }}
                  animate={{ opacity: 1, filter: 'blur(0px)', scale: 1,    y: 0  }}
                  exit={{   opacity: 0, filter: 'blur(4px)', scale: 0.95, y: 16  }}
                  transition={{ type: 'spring', duration: 0.4, bounce: 0 }}
                >
                  <h2 style={{ color: '#fff' }}>{selectedItem.title}</h2>
                  <p style={{ color: 'rgba(255,255,255,0.8)' }}>{selectedItem.description}</p>
                  <button onClick={() => setSelected(null)}>Close</button>
                </motion.div>
              </AnimatePresence>
            </motion.div>
          </>
        )}
      </AnimatePresence>
    </>
  );
}
```

**Example — tab indicator (minimal use case):**

```tsx
// The indicator is a separate element that morphs between tab positions
// Not the tab itself — just the underline/pill

{tabs.map(tab => (
  <button key={tab} onClick={() => setActive(tab)} style={{ position: 'relative' }}>
    {tab}
    {active === tab && (
      <motion.div
        layoutId="tab-indicator"   // same id across all tabs = slides between them
        style={{
          position: 'absolute', bottom: -2,
          left: 0, right: 0, height: 2,
          background: 'currentColor',
        }}
        transition={{ type: 'spring', stiffness: 400, damping: 30 }}
      />
    )}
  </button>
))}
```

**Gotchas:**
- `layoutId` must be unique across the entire component tree — two unrelated groups of elements cannot share the same id
- If both the source and target render simultaneously (e.g. a tab indicator where all tabs are always mounted), use the conditional render pattern above so only one instance exists at a time
- If both must be in the DOM simultaneously, use the `layout` prop instead of `layoutId`
- `border-radius` animates automatically during layout transitions — no need to manually animate it

**When to apply:**
- Card grid → expanded modal or detail view
- Active tab / nav indicator sliding between items
- Chip or tag moving between "available" and "selected" lists
- Any element that represents the same object in two different visual states

---

## 4. Page / Route Transition

Without transitions, page navigation feels like a hard cut. Even a simple fade-through with slight movement adds a sense of spatial continuity.

```tsx
// Works with Next.js App Router, React Router, or any router
// Wrap page content — not the layout shell

import { AnimatePresence, motion } from 'motion/react';
import { usePathname } from 'next/navigation'; // or useLocation in React Router

const pageVariants = {
  initial: { opacity: 0, y: 8, filter: 'blur(4px)' },
  animate: { opacity: 1, y: 0, filter: 'blur(0px)' },
  exit:    { opacity: 0, y: -8, filter: 'blur(4px)' },
};

const pageTransition = {
  duration: 0.3,
  ease: [0.19, 1, 0.22, 1],
};

// In your layout / router outlet:
export function PageTransition({ children }) {
  const pathname = usePathname();

  return (
    <AnimatePresence mode="wait">
      <motion.div
        key={pathname}           // ← key change triggers exit + enter
        variants={pageVariants}
        initial="initial"
        animate="animate"
        exit="exit"
        transition={pageTransition}
      >
        {children}
      </motion.div>
    </AnimatePresence>
  );
}
```

**Gotcha:** `mode="wait"` makes the exiting page finish its exit before the entering page starts. Use `mode="sync"` if you want them to overlap (crossfade). `mode="wait"` is safer for most layouts.

**When to apply:**
- Any multi-page app or router-driven navigation
- Tab panels with distinct content
- Wizard / multi-step flows where each step is a distinct view

---

## 5. List Reorder Animation

When items in a list change order — drag-and-drop, sort by column, filter results — they teleport to their new positions. `layout` prop makes Motion animate each item to its new position automatically.

```tsx
import { Reorder, motion } from 'motion/react';
import { useState } from 'react';

// Option A — drag-to-reorder with Reorder component
export function DraggableList({ initialItems }) {
  const [items, setItems] = useState(initialItems);

  return (
    <Reorder.Group axis="y" values={items} onReorder={setItems}>
      {items.map(item => (
        <Reorder.Item key={item.id} value={item}>
          <div>{item.label}</div>
        </Reorder.Item>
      ))}
    </Reorder.Group>
  );
}

// Option B — programmatic reorder (sort, filter) with layout animation
export function SortableList({ items }) {
  return (
    <ul>
      {items.map(item => (
        <motion.li
          key={item.id}
          layout                          // ← animates to new position
          transition={{ type: 'spring', stiffness: 300, damping: 30 }}
        >
          {item.label}
        </motion.li>
      ))}
    </ul>
  );
}
```

**Gotcha:** `layout` on list items only works if the `key` stays stable across renders. If you're using array index as key, reordering will re-mount items rather than animate them.

**When to apply:**
- Drag-and-drop reorderable lists
- Sort-by-column in tables or card grids
- Filter results where items shift position
- Kanban board column reordering

---

## 6. Mode Switching

Tab content, toggle views (list/grid), before/after comparisons. When the active mode changes, the outgoing content should exit before or alongside the incoming content — not just be replaced.

```tsx
import { AnimatePresence, motion } from 'motion/react';
import { useState } from 'react';

const views = ['List', 'Grid', 'Map'];

export function ModeSwitch({ renderView }) {
  const [active, setActive] = useState('List');

  return (
    <div>
      {/* Tab bar with sliding indicator */}
      <div style={{ position: 'relative', display: 'flex' }}>
        {views.map(view => (
          <button key={view} onClick={() => setActive(view)}>
            {view}
            {active === view && (
              <motion.div
                layoutId="active-tab-indicator"   // slides between tabs
                style={{
                  position: 'absolute', bottom: 0,
                  height: 2, background: 'currentColor',
                  width: '100%', left: 0,
                }}
                transition={{ type: 'spring', stiffness: 400, damping: 30 }}
              />
            )}
          </button>
        ))}
      </div>

      {/* Content area — coordinated exit + enter */}
      <AnimatePresence mode="wait">
        <motion.div
          key={active}
          initial={{ opacity: 0, y: 6 }}
          animate={{ opacity: 1, y: 0 }}
          exit={{ opacity: 0, y: -6 }}
          transition={{ duration: 0.2, ease: 'easeOut' }}
        >
          {renderView(active)}
        </motion.div>
      </AnimatePresence>
    </div>
  );
}
```

**When to apply:**
- Tab panels
- List / grid / map view toggles
- Settings sections with distinct content per selection
- Before/after or compare modes

---

## 7. Drag with Snap-Back

A draggable element that teleports back to its origin on release looks broken. It should spring back with physical weight — fast departure from the drag position, natural settle at origin.

```tsx
import { motion, useMotionValue, useTransform, animate } from 'motion/react';

export function DraggableCard({ children }) {
  const x = useMotionValue(0);
  const y = useMotionValue(0);

  // Subtle rotation based on drag distance — adds physicality
  const rotate = useTransform(x, [-200, 200], [-12, 12]);
  const opacity = useTransform(
    x,
    [-200, -100, 0, 100, 200],
    [0.4, 0.8, 1, 0.8, 0.4]
  );

  function handleDragEnd() {
    // Spring back to origin
    animate(x, 0, { type: 'spring', stiffness: 300, damping: 25 });
    animate(y, 0, { type: 'spring', stiffness: 300, damping: 25 });
  }

  return (
    <motion.div
      style={{ x, y, rotate, opacity }}
      drag
      dragConstraints={{ left: 0, right: 0, top: 0, bottom: 0 }} // soft constraints
      dragElastic={0.15}   // how much it resists dragging past constraints
      onDragEnd={handleDragEnd}
      whileDrag={{ cursor: 'grabbing', scale: 1.02 }}
    >
      {children}
    </motion.div>
  );
}
```

**When to apply:**
- Swipeable cards (onboarding, flashcards, decision flows)
- Draggable UI elements that return to a fixed position
- Any drag interaction where the element shouldn't stay where it's dropped

---

## 8. Swipe to Dismiss

List items or cards that can be removed via swipe gesture. The item should exit in the direction of the swipe, and the remaining items should animate into the vacated space.

```tsx
import { AnimatePresence, motion, useMotionValue, useTransform } from 'motion/react';
import { useState } from 'react';

function SwipeItem({ id, label, onDismiss }) {
  const x = useMotionValue(0);
  const opacity = useTransform(x, [-120, -60, 0, 60, 120], [0, 0.5, 1, 0.5, 0]);
  const background = useTransform(
    x,
    [-120, 0, 120],
    ['#ef4444', '#ffffff', '#22c55e']
  );

  function handleDragEnd(_, info) {
    if (Math.abs(info.offset.x) > 100) {
      onDismiss(id); // threshold exceeded — dismiss
    }
  }

  return (
    <motion.div
      style={{ x, opacity }}
      drag="x"
      dragConstraints={{ left: 0, right: 0 }}
      dragElastic={0.2}
      onDragEnd={handleDragEnd}
      exit={{ x: 200, opacity: 0, transition: { duration: 0.2 } }}
      layout  // ← remaining items animate into the gap
    >
      {label}
    </motion.div>
  );
}

export function SwipeList({ initialItems }) {
  const [items, setItems] = useState(initialItems);

  const dismiss = (id) => setItems(items => items.filter(i => i.id !== id));

  return (
    <AnimatePresence>
      {items.map(item => (
        <SwipeItem key={item.id} {...item} onDismiss={dismiss} />
      ))}
    </AnimatePresence>
  );
}
```

**Gotcha:** The `layout` prop on each item is what makes remaining items slide up smoothly after a dismissal. Without it they jump.

**When to apply:**
- Notification lists
- Email / inbox item removal
- Task or to-do list items
- Mobile card stacks

---

## 9. Press Feedback

Buttons that feel flat give no tactile signal that the interaction registered. A subtle scale response on press closes the feedback loop — the UI confirms the action before the server does.

```tsx
import { motion } from 'motion/react';

// Minimal — scale on press
export function PressButton({ children, onClick }) {
  return (
    <motion.button
      whileTap={{ scale: 0.96 }}
      transition={{ type: 'spring', stiffness: 400, damping: 20 }}
      onClick={onClick}
    >
      {children}
    </motion.button>
  );
}

// With hover state — full interactive feedback
export function InteractiveButton({ children, onClick, variant = 'primary' }) {
  return (
    <motion.button
      whileHover={{ scale: 1.02, y: -1 }}
      whileTap={{ scale: 0.97, y: 0 }}
      transition={{ type: 'spring', stiffness: 400, damping: 20 }}
      onClick={onClick}
    >
      {children}
    </motion.button>
  );
}

// Icon button — more pronounced, smaller target
export function IconButton({ icon, onClick, label }) {
  return (
    <motion.button
      aria-label={label}
      whileHover={{ scale: 1.1 }}
      whileTap={{ scale: 0.9, rotate: -5 }}
      transition={{ type: 'spring', stiffness: 500, damping: 20 }}
      onClick={onClick}
    >
      {icon}
    </motion.button>
  );
}
```

**Scale reference by button type:**
- Primary / large buttons: `whileTap: { scale: 0.96–0.97 }`
- Secondary / medium buttons: `whileTap: { scale: 0.95–0.96 }`
- Icon buttons: `whileTap: { scale: 0.88–0.92 }`

**When to apply:**
- All clickable buttons — this is the baseline bar
- Icon-only actions (close, like, share, delete)
- Card-level click interactions

**Anti-pattern:** `whileTap: { scale: 0.85 }` on a large button — too dramatic, reads as broken rather than tactile.

---

## 10. Skeleton → Content Transition

When real content replaces a skeleton, the swap should feel like a reveal — not a hard cut. The skeleton fades out as the content fades in, occupying the same space.

```tsx
import { AnimatePresence, motion } from 'motion/react';

function SkeletonLine({ width = '100%' }) {
  return (
    <motion.div
      style={{
        width, height: 16, borderRadius: 4,
        background: 'var(--color-muted)',
      }}
      animate={{ opacity: [0.5, 1, 0.5] }}
      transition={{ duration: 1.5, repeat: Infinity, ease: 'easeInOut' }}
    />
  );
}

export function ContentCard({ isLoading, data }) {
  return (
    <div style={{ position: 'relative', minHeight: 120 }}>
      <AnimatePresence mode="wait">
        {isLoading ? (
          <motion.div
            key="skeleton"
            exit={{ opacity: 0 }}
            transition={{ duration: 0.2 }}
            style={{ display: 'flex', flexDirection: 'column', gap: 8 }}
          >
            <SkeletonLine width="60%" />
            <SkeletonLine width="100%" />
            <SkeletonLine width="80%" />
          </motion.div>
        ) : (
          <motion.div
            key="content"
            initial={{ opacity: 0, filter: 'blur(4px)' }}
            animate={{ opacity: 1, filter: 'blur(0px)' }}
            transition={{ duration: 0.3, ease: 'easeOut' }}
          >
            <h3>{data.title}</h3>
            <p>{data.description}</p>
          </motion.div>
        )}
      </AnimatePresence>
    </div>
  );
}
```

**Gotcha:** Give the container a `minHeight` that approximates the content height. Otherwise the layout jumps when the skeleton disappears and the content hasn't fully measured yet.

**When to apply:**
- Data-fetching cards and panels
- User profile sections
- Any component with an async loading state

---

## 11. Optimistic UI Animation

The UI shouldn't wait for the server to confirm before giving feedback. Update state immediately, animate the response, revert if the request fails.

```tsx
import { motion, AnimatePresence } from 'motion/react';
import { useState } from 'react';

export function LikeButton({ postId, initialLiked, initialCount }) {
  const [liked, setLiked] = useState(initialLiked);
  const [count, setCount] = useState(initialCount);
  const [isReverting, setIsReverting] = useState(false);

  async function handleLike() {
    const wasLiked = liked;
    const prevCount = count;

    // Immediate optimistic update
    setLiked(!wasLiked);
    setCount(c => wasLiked ? c - 1 : c + 1);

    try {
      await api.toggleLike(postId);
    } catch {
      // Revert on failure
      setIsReverting(true);
      setLiked(wasLiked);
      setCount(prevCount);
      setTimeout(() => setIsReverting(false), 500);
    }
  }

  return (
    <button onClick={handleLike}>
      <motion.span
        animate={liked
          ? { scale: [1, 1.4, 1], rotate: [0, -15, 10, 0] }
          : { scale: 1 }
        }
        transition={{ duration: 0.4, ease: 'easeOut' }}
      >
        {liked ? '❤️' : '🤍'}
      </motion.span>

      {/* Animated count — see Pattern 12 for number animation */}
      <AnimatePresence mode="popLayout">
        <motion.span
          key={count}
          initial={{ y: liked ? 8 : -8, opacity: 0 }}
          animate={{ y: 0, opacity: 1 }}
          exit={{ y: liked ? -8 : 8, opacity: 0 }}
          transition={{ duration: 0.2 }}
        >
          {count}
        </motion.span>
      </AnimatePresence>

      {/* Revert indicator */}
      <AnimatePresence>
        {isReverting && (
          <motion.span
            initial={{ opacity: 0, y: -4 }}
            animate={{ opacity: 1, y: 0 }}
            exit={{ opacity: 0 }}
            style={{ fontSize: 11, color: '#ef4444' }}
          >
            Failed
          </motion.span>
        )}
      </AnimatePresence>
    </button>
  );
}
```

**When to apply:**
- Like / bookmark / follow actions
- Toggle states backed by an API
- Form autosave
- Any mutation where the happy path is > 95% likely

---

## 12. Number Counter Animation

Numbers that update instantly — live metrics, counters, scores — give no sense of the magnitude of change. Animating the transition from old to new value communicates direction and scale.

```tsx
import { useSpring, useTransform, motion, animate } from 'motion/react';
import { useEffect, useRef } from 'react';

// Option A — animate the displayed number directly
export function AnimatedNumber({ value, format = (n: number) => Math.round(n).toLocaleString() }) {
  const ref = useRef<HTMLSpanElement>(null);
  const motionValue = useSpring(value, { stiffness: 100, damping: 30 });

  useEffect(() => {
    motionValue.set(value);
  }, [value, motionValue]);

  useEffect(() => {
    return motionValue.on('change', (latest) => {
      if (ref.current) {
        ref.current.textContent = format(latest);
      }
    });
  }, [motionValue, format]);

  return (
    <span
      ref={ref}
      className="tabular-nums" // always pair with tabular-nums
    >
      {format(value)}
    </span>
  );
}

// Option B — slot machine style (digits scroll in/out)
export function SlotNumber({ value }: { value: number }) {
  return (
    <AnimatePresence mode="popLayout">
      <motion.span
        key={value}
        initial={{ y: 12, opacity: 0, filter: 'blur(4px)' }}
        animate={{ y: 0,  opacity: 1, filter: 'blur(0px)' }}
        exit={{   y: -12, opacity: 0, filter: 'blur(4px)' }}
        transition={{ type: 'spring', stiffness: 300, damping: 25 }}
        className="tabular-nums"
        style={{ display: 'inline-block' }}
      >
        {value.toLocaleString()}
      </motion.span>
    </AnimatePresence>
  );
}
```

**Option A vs B:**
- Option A (spring count-up): best for large value jumps, dashboards, analytics. Communicates scale of change.
- Option B (slot machine): best for small increments, counters, scores. More playful, lower overhead.

**Always pair with `tabular-nums`** — the digit width must stay fixed or the layout shifts as the number animates.

**When to apply:**
- Live metric dashboards
- Score counters
- Cart item counts
- Notification badge numbers
- Price updates

---

## 13. Progress Bar with Spring

A progress bar that jumps to its new value looks mechanical. A spring-driven bar feels like it has momentum — it overshoots slightly and settles, giving a sense of physical weight to the progress.

```tsx
import { motion, useSpring, useTransform } from 'motion/react';
import { useEffect } from 'react';

export function SpringProgress({ value, max = 100, height = 6 }) {
  const percentage = (value / max) * 100;
  const spring = useSpring(percentage, { stiffness: 80, damping: 20 });

  useEffect(() => {
    spring.set(percentage);
  }, [percentage, spring]);

  const width = useTransform(spring, v => `${v}%`);

  return (
    <div
      style={{
        width: '100%',
        height,
        borderRadius: height / 2,
        background: 'var(--color-muted)',
        overflow: 'hidden',
      }}
    >
      <motion.div
        style={{
          width,
          height: '100%',
          borderRadius: height / 2,
          background: 'var(--color-primary)',
          transformOrigin: 'left',
        }}
      />
    </div>
  );
}

// Usage
<SpringProgress value={uploadProgress} max={100} />
<SpringProgress value={completedTasks} max={totalTasks} height={8} />
```

**Tuning the spring:**
- `stiffness: 80, damping: 20` — gentle, noticeable overshoot. Good for file upload, onboarding progress.
- `stiffness: 120, damping: 28` — snappier, minimal overshoot. Good for form completion, step indicators.
- `stiffness: 200, damping: 40` — fast, almost no overshoot. Good for real-time metrics.

**When to apply:**
- File upload progress
- Onboarding / setup completion
- Form step indicators
- Any progress bar that updates incrementally

---

## Shared Utilities

These are used across multiple patterns above. Define once, import everywhere.

```tsx
// hooks/useMeasure.ts
import { useCallback, useEffect, useState } from 'react';

export function useMeasure<T extends HTMLElement = HTMLElement>() {
  const [element, setElement] = useState<T | null>(null);
  const [bounds, setBounds] = useState({ width: 0, height: 0 });

  const ref = useCallback((node: T | null) => setElement(node), []);

  useEffect(() => {
    if (!element) return;
    const observer = new ResizeObserver(([entry]) => {
      setBounds({
        width: entry.contentRect.width,
        height: entry.contentRect.height,
      });
    });
    observer.observe(element);
    return () => observer.disconnect();
  }, [element]);

  return [ref, bounds] as const;
}
```

```tsx
// lib/motion.ts — shared transition presets
export const transitions = {
  snappy:    { type: 'spring', stiffness: 400, damping: 28 } as const,
  physical:  { type: 'spring', stiffness: 260, damping: 20 } as const,
  gentle:    { type: 'spring', stiffness: 120, damping: 20 } as const,
  swift:     { duration: 0.3, ease: [0.19, 1, 0.22, 1] }   as const,
  smooth:    { duration: 0.4, ease: [0.4, 0, 0.2, 1] }     as const,
};
```