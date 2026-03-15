---
description: Apply Laws of UX to the current component or screen. Audits for usability violations — hit area size, decision complexity, information chunking, perceived performance, and input tolerance — and fixes them with concrete code changes.
globs: ["**/*.tsx", "**/*.jsx", "**/*.ts", "**/*.css"]
---

# Laws of UX

Reference command distilled from [Laws of UX](https://www.userinterface.wiki/laws-of-ux).

When invoked, scan the current component or file for violations of the laws below and fix them. For each fix, call out what was wrong and why it matters.

---

## Quick Reference

| Law | Trigger Condition | Fix |
|---|---|---|
| Fitts's Law | Interactive elements with no padding | Expand hit area with `::before` or padding |
| Hick's Law | 7+ options visible at once | Group, collapse, progressively disclose |
| Miller's Law | Raw unformatted numbers/IDs in UI | Chunk and format before rendering |
| Doherty Threshold | Network actions with no instant feedback | Optimistic UI, skeletons, progress |
| Postel's Law | Strict input validation rejecting valid formats | Normalize input, validate generously |

---

## 1. Fitts's Law — Make Targets Easy to Hit

The time to reach a target increases as it gets smaller or farther away. Every pixel of padding on an interactive element is a usability decision.

**Rule:** Always think about the hit area of interactive elements, not just their visual size.

```css
/* Expand hit area without changing visual size */
.btn {
  position: relative;
}

.btn::before {
  content: '';
  position: absolute;
  inset: -8px;
}
```

```tsx
// In React — use padding to expand hit area visually or invisibly
<button className="relative p-3 before:absolute before:inset-[-8px]">
  <Icon />
</button>
```

**When to apply:**
- Icon-only buttons (close, expand, delete)
- Small checkboxes or radio inputs
- Links inside dense lists or tables
- Mobile touch targets (minimum 44×44px recommended)

**Anti-pattern:** Relying on the visual size alone — a 16px icon with no padding is nearly impossible to tap on mobile.

---

## 2. Hick's Law — Reduce Decision Complexity

The time to make a decision grows logarithmically with the number of choices. More options = more cognitive load.

**Rule:** Show only what matters now. Use progressive disclosure to reveal complexity when needed.

**Audit trigger — count, then act:**
- Count direct children of any `<nav>`, `<menu>`, settings list, or dropdown
- If count > 7: group the excess items under a collapsible section or secondary nav group
- If a dropdown has > 7 items and no search input: add a search/filter input
- If an onboarding or wizard step contains more than 1 primary decision: split into separate steps

```tsx
// Violation — 8 flat nav items, count exceeds 7
<nav>
  <NavItem>Dashboard</NavItem>
  <NavItem>Analytics</NavItem>
  <NavItem>Reports</NavItem>
  <NavItem>Settings</NavItem>
  <NavItem>Integrations</NavItem>
  <NavItem>Billing</NavItem>
  <NavItem>Team</NavItem>
  <NavItem>Audit Log</NavItem>
</nav>

// Fix — group items beyond the first 4–5 under a labelled section
<nav>
  <NavItem>Dashboard</NavItem>
  <NavItem>Analytics</NavItem>
  <NavItem>Reports</NavItem>
  <NavGroup label="Settings">
    <NavItem>Integrations</NavItem>
    <NavItem>Billing</NavItem>
    <NavItem>Team</NavItem>
    <NavItem>Audit Log</NavItem>
  </NavGroup>
</nav>
```

**When to apply:**
- Any nav, menu, or list with more than 7 direct children
- Dropdowns with more than 7 options and no search
- Settings pages where all options are rendered flat with no grouping
- Onboarding steps with more than one primary action or decision

**Anti-pattern:** Showing all features upfront to demonstrate capability — it overwhelms rather than impresses.

---

## 3. Miller's Law — Chunk Information

The average person can hold ~7 (±2) items in working memory at once. Raw data blobs are the enemy.

**Rule:** Group and chunk data so the brain can process it without strain.

```tsx
// Bad — raw unformatted data
<span>14155552671</span>
<span>4111111111111111</span>

// Good — chunked for readability
<span>+1 (415) 555-2671</span>
<span>4111 1111 1111 1111</span>
```

```tsx
// Utility: chunk a string every N characters
const chunk = (str: string, size: number) =>
  str.match(new RegExp(`.{1,${size}}`, 'g'))?.join(' ') ?? str;

// Phone formatting
const formatPhone = (val: string) =>
  val.replace(/(\d{3})(\d{3})(\d{4})/, '($1) $2-$3');

// Credit card formatting
const formatCard = (val: string) => chunk(val.replace(/\s/g, ''), 4);
```

**When to apply:**
- Phone numbers, card numbers, IBANs, IDs
- Long lists — break into sections with headers
- Tables — group rows, add visual separators every 5–7 rows
- Dense forms — split into logical sections, not one long scroll

**Anti-pattern:** Displaying raw API data directly in the UI — always format before rendering.

---

## 4. Doherty Threshold — Keep Interactions Under 400ms

Under 400ms feels instant. Above it, users notice. Above 2000ms, they assume something is broken.

**Rule:** If you can't make it fast, make it *feel* fast. Use optimistic UI, skeleton screens, and progress indicators.

**Audit trigger — scan for these exact conditions:**
- Any `<button>` or clickable element with an `async` handler that sets no state before `await` → add optimistic state update before the `await` call
- Any component with an `isLoading` prop or state that renders `null`, a spinner, or blank space → replace with a skeleton that matches the content's shape
- Any data-fetching component (uses `fetch`, `axios`, `useQuery`, `useSWR` etc.) with no loading state at all → add skeleton
- Any `<button>` that gets `disabled` on click with no visible loading indicator → add inline label change or spinner before disabling

```tsx
// Violation — button disabled on click, no feedback
<button disabled={isSubmitting} onClick={handleSubmit}>
  Submit
</button>

// Fix — immediate label change visible before await resolves
<button disabled={isSubmitting} onClick={handleSubmit}>
  {isSubmitting ? 'Submitting...' : 'Submit'}
</button>
```

```tsx
// Violation — renders nothing while loading
if (isLoading) return null;

// Fix — skeleton matching content shape
if (isLoading) return (
  <div className="animate-pulse space-y-2">
    <div className="h-4 bg-muted rounded w-3/4" />
    <div className="h-4 bg-muted rounded w-1/2" />
  </div>
);
```

```tsx
// Optimistic UI — update state before the request resolves
const handleLike = async (id: string) => {
  setLiked(true); // immediate — before await
  try {
    await api.like(id);
  } catch {
    setLiked(false); // revert on failure
  }
};
```

**Anti-pattern:** `disabled` on click with no label or spinner change — users assume nothing happened and click again.

---

## 5. Postel's Law — Accept Messy Input, Output Clean Data

Be liberal in what you accept, be strict in what you output. Users think in meaning, not format.

**Rule:** Normalize input on the way in. Never force users to think about formatting.

```tsx
// Accept any date format, normalize to ISO
const parseDate = (input: string): string | null => {
  const cleaned = input.trim().toLowerCase();
  const date = new Date(cleaned);
  if (!isNaN(date.getTime())) return date.toISOString().split('T')[0];
  return null;
};

// Usage
parseDate('jan 15 2024')   // → '2024-01-15'
parseDate('15/01/2024')    // → '2024-01-15'
parseDate('2024-01-15')    // → '2024-01-15'
```

```tsx
// Accept phone with any formatting, strip to digits
const normalizePhone = (input: string) =>
  input.replace(/\D/g, '');

// Accept card number with spaces or dashes
const normalizeCard = (input: string) =>
  input.replace(/[\s-]/g, '');
```

**When to apply:**
- Date inputs — accept natural language, relative dates, any delimiter
- Phone inputs — accept spaces, dashes, parentheses, country codes
- Search — be fuzzy, ignore case, ignore punctuation
- Paste events — strip formatting automatically

**Anti-pattern:** Showing a validation error for `(415) 555-2671` when you only accept `4155552671` — the user is right, the interface is wrong.

---

## Summary

| Law | Core Rule | Key Technique |
|---|---|---|
| Fitts's Law | Make targets easy to hit | Expand hit areas with `::before` / padding |
| Hick's Law | Reduce choices shown at once | Progressive disclosure, grouping |
| Miller's Law | Chunk data into groups of ~7 | Format numbers, group lists, section forms |
| Doherty Threshold | Keep responses under 400ms | Optimistic UI, skeletons, prefetching |
| Postel's Law | Accept messy input, output clean data | Normalize on input, validate generously |