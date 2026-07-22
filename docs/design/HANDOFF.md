# From Penpot to React + Tailwind — implementation handoff

Written for a learner who has never opened Penpot. The design file is the
pixel spec; this page tells you how to read it and how its rules become
code. The deeper reference is [DESIGN.md](DESIGN.md); the boards match the
screenshots in [`screens/`](screens/).

---

## 1. Reading the Penpot file

- **Boards are screens.** Top row = reusable components; below that, one
  row per role (public, mentee, mentor, staff, mobile). Desktop boards are
  1280 wide, mobile 375. Everything in between the two widths is your
  responsibility with responsive utilities.
- **Measure by selecting.** Click a shape, hold <kbd>Alt</kbd> and hover
  another to see the distance between them. The right sidebar shows size,
  fills, strokes, and radius.
- **Colors live in the Tokens tab** (left sidebar). Read color *names*
  there (`color.accent`, `color.line`…) — never copy a hex from the color
  picker into your code (§2 explains why).
- **Theme switch:** Tokens tab → Themes dropdown → Mode / Light or Dark.
  Use only that dropdown — don't tick the `light`/`dark` entries in the
  Sets list; that wedges the file (see PENPOT-BUILD.md for recovery).
- **Icons and the thread** (dot–curve–dot) are vectors: right-click →
  Export to get SVG.
- ⚠️ Penpot can "copy as CSS", and it's useful to *read* a value — but the
  output is absolute-positioned pixel soup. Never paste it.

## 2. Colors: semantic tokens, never hex

There are exactly 14 colors. In code they are CSS variables consumed
through Tailwind, so you write `bg-surface text-fg border-line` — never
`#faf6f2`. Dark mode is **the same classes with different variable
values**, switched by `data-theme` on `<html>` (plus a
`prefers-color-scheme` fallback). If you hardcode a hex, dark mode breaks.

| Token | Tailwind class | Use for |
|---|---|---|
| `bg` | `bg-bg` | page background |
| `fg` | `text-fg` | body text |
| `surface` | `bg-surface` | cards, inputs, header |
| `line` | `border-line` | borders, dividers, off switch track |
| `muted` | `text-muted` | secondary text |
| `accent` | `bg-accent` / `text-accent` | ALL interactive color |
| `accent-hover` | `hover:bg-accent-hover` | hover of accent |
| `accent-soft` | decoration only | thread, underlines, big numerals — **never small text** |
| `tint` | `bg-tint` | active nav pill, selected role card, soft notices |
| `on-accent` | `text-on-accent` | text sitting on an `accent` fill — never raw white |
| `ok` / `ok-tint` | `text-ok` / `bg-ok-tint` | success text / pill fill |
| `warn` / `warn-tint` | `text-warn` / `bg-warn-tint` | warnings |

Setup is Tailwind v4, CSS-first — no `tailwind.config.js`. In
`index.css`: define the variables on `:root`, override them under
`:root[data-theme="dark"]` and the `prefers-color-scheme: dark` media
query, then expose them with `@theme inline`:

```css
@import "tailwindcss";
@import "@fontsource-variable/fraunces/index.css";
@import "@fontsource-variable/public-sans/index.css";

:root { --bg: #faf6f2; --fg: #231d1b; /* …see DESIGN.md §2 for all 14 */ }
:root[data-theme="dark"] { --bg: #191412; /* …dark values */ }

@theme inline {
  --color-bg: var(--bg);
  --color-fg: var(--fg);
  /* …one line per token: makes bg-bg, text-fg, etc. exist */
  --font-display: "Fraunces Variable", Georgia, serif;
  --font-sans: "Public Sans Variable", Arial, sans-serif;
}
```

The complete, working version of this file — all 14 tokens, both dark
blocks (OS preference *and* explicit toggle), the paper-grain overlay,
the thread/rise animations with their `prefers-reduced-motion` reset, and
the `.overshoot` underline — is the app's own
[`frontend/src/index.css`](../../frontend/src/index.css). Start by
copying it verbatim; it is the design system in code form. When you add
a token, it goes in three places: `:root`, the two dark blocks, and
`@theme inline`.

## 3. Type: two families, eight styles

**Fraunces SemiBold** (`font-display font-semibold`) for every heading,
stat numeral, and person name. **Public Sans** (`font-sans`) for
everything else — Regular for prose, SemiBold for buttons, labels, links.
Self-hosted via `@fontsource-variable`, no CDN.

| Style | Face | Size/line | Notes |
|---|---|---|---|
| Hero | Fraunces SB | 56–60 / 1.05 | landing h1, tracking −2% |
| H1 | Fraunces SB | 36 / 1.15 | page titles |
| H2 | Fraunces SB | 30 / 1.2 | section heads |
| Card title | Fraunces SB | 20 / 1.3 | names, empty states |
| Body | Public Sans R | 17 / 1.65 | prose (not `text-base`!) |
| Small | Public Sans R | 14 / 1.5 | table cells |
| Small SB | Public Sans SB | 14 / 1.4 | buttons, links, nav |
| Eyebrow | Public Sans SB | 12 / 1.3 | uppercase, tracking +9–14% |

H1/H2 carry the **overshoot underline**: a 3px `accent-soft` bar, radius
2, extending ~0.15em past the left edge and ~1.2em past the right — an
`::after` element, not a border (`.overshoot` in the app's `index.css`).

## 4. Layout constants

- Content column: `max-w-6xl` (1152) + `px-4`; forms `max-w-sm` (384).
- Header: sticky, 64px, `bg` at 90% + backdrop-blur, 1px `line` bottom.
- 40px between page sections; landing bands use ~96px vertical padding.
- Mobile `< 768px` (`md:`): nav collapses to a hamburger opening a 288px
  right slide-over drawer (`surface`, left `line` border, Escape and
  backdrop close it).
- Wide tables scroll inside their own `overflow-x-auto` container — the
  page never scrolls sideways.

## 5. Worked examples (the pattern to copy)

**Button** — radius 6, min-height 44, padding 10×16, Small SB. The
variants differ only in color classes, so it's one component + a prop:

```jsx
const styles = {
  primary: "bg-accent text-on-accent hover:bg-accent-hover",
  outline: "border-[1.5px] border-accent text-accent hover:bg-tint",
  quiet:   "text-muted hover:bg-tint",
};

export function Button({ variant = "primary", className = "", ...props }) {
  return (
    <button
      className={`min-h-11 rounded-md px-4 py-2.5 font-sans text-sm font-semibold
        transition-colors focus-visible:outline focus-visible:outline-2
        focus-visible:outline-offset-2 focus-visible:outline-accent
        ${styles[variant]} ${className}`}
      {...props}
    />
  );
}
```

**Status pill** — the four visual states map 1:1 to match states in the
data, exactly like the variant set in Penpot. Drive it with data, never
with copy-pasted markup:

```jsx
const STATUS = {
  proposed: { classes: "bg-tint text-accent",         label: "Awaiting acceptance" },
  accepted: { classes: "bg-warn-tint text-warn",      label: "Chemistry & confirm" },
  active:   { classes: "bg-ok-tint text-ok",          label: "Active" },
  closed:   { classes: "bg-line/60 text-muted",       label: "Closed" },
};

export function StatusPill({ status }) {
  const s = STATUS[status];
  return (
    <span className={`rounded-full px-2.5 py-1 text-xs font-semibold ${s.classes}`}>
      {s.label}
    </span>
  );
}
```

## 6. What the boards can't show you

- **Hover:** primary buttons darken to `accent-hover`; outline buttons and
  chips gain `tint` fill / `accent` border; links shift underline color.
- **Focus:** every interactive element gets the 2px `accent` outline with
  2px offset (`focus-visible:` in the Button above). Never remove it.
- **Motion:** ONE orchestrated moment — the landing hero lines rise
  staggered (0.55s, 50–400ms delays) and the thread draws itself in 0.9s
  via `stroke-dashoffset`. Everything else is `transition-colors`.
  `prefers-reduced-motion` disables all of it — no exceptions.
- Transient states (save toasts, "no compatible mentor") aren't captured;
  their copy lives in the source (`MenteeDashboard.tsx`, `Profile.tsx`,
  `StaffDashboard.tsx`).

## 7. Semantics the design implies

- Discipline/availability **chips are toggle buttons** with
  `aria-pressed`, not checkboxes.
- The **switch** is `role="switch"` + `aria-checked`, track 44×24, 20px
  white knob translating 20px.
- The **journey rail** is a list; "done" labels are struck-through text,
  not an image.
- Touch targets ≥ 44px (`min-h-11`); chips and buttons already are.
- `accent` is the *only* interactive color; `ok`/`warn` mark states, never
  decoration.

## 8. Content rules

Copy text **verbatim** from the boards — it is the product voice. All
names in the design (Sam Mensah, Bola Proposed, Ruta Radiya…) are
fictional stand-ins; real data comes from the API.

## Definition of done (per screen)

1. Side-by-side with the matching PNG in `screens/` at 1280 — spacing and
   type sizes match.
2. Toggle `data-theme="dark"` — nothing is hardcoded, everything flips.
3. Tab through the screen — every control shows the focus ring.
4. 375px viewport — matches the mobile board; no horizontal scroll.
5. Zero raw hex values and zero raw `white`/`black` in your diff.

---

## Appendix: the complete `index.css`, annotated

The full stylesheet, block by block. This is the design system in code
form — everything else in the app is Tailwind utility classes built on
top of it.

```css
@import "tailwindcss";
@import "@fontsource-variable/fraunces/index.css";
@import "@fontsource-variable/public-sans/index.css";

/* ---------- 1. Semantic tokens: light (default) ---------- */
:root {
  color-scheme: light;      /* native controls (selects, scrollbars) match */
  --bg: #faf6f2;            /* warm paper */
  --fg: #231d1b;            /* warm ink */
  --surface: #ffffff;
  --line: #e9ddd5;
  --muted: #6f6259;
  --accent: #d80f0f;        /* CYF red — AA on paper/white */
  --accent-hover: #a71000;
  --accent-soft: #ea4549;   /* decoration only — never small text */
  --tint: #fbeae7;
  --on-accent: #ffffff;
  --ok: #1f7a4d;
  --ok-tint: #e7f3ec;
  --warn: #875c15;
  --warn-tint: #f8efdd;
}

/* ---------- 2. Dark values, twice ----------
   (a) OS prefers dark AND the user hasn't forced light;
   (b) the user explicitly chose dark via the toggle.
   Both blocks are identical — that duplication is what makes every
   OS-theme × app-toggle combination work. Miss one and some
   combination shows the wrong theme. */
@media (prefers-color-scheme: dark) {
  :root:not([data-theme="light"]) {
    color-scheme: dark;
    --bg: #191412;  --fg: #f2eae4;  --surface: #251e1a;
    --line: #3d322c; --muted: #a99a90;
    --accent: #f5807f;        /* lifted for contrast on dark */
    --accent-hover: #ffa19f;  --accent-soft: #ea4549;
    --tint: #38221f; --on-accent: #1c1210;
    --ok: #79cfa0;  --ok-tint: #22352b;
    --warn: #dcaa54; --warn-tint: #3a2f1c;
  }
}
:root[data-theme="dark"] {
  color-scheme: dark;
  --bg: #191412;  --fg: #f2eae4;  --surface: #251e1a;
  --line: #3d322c; --muted: #a99a90;
  --accent: #f5807f; --accent-hover: #ffa19f; --accent-soft: #ea4549;
  --tint: #38221f; --on-accent: #1c1210;
  --ok: #79cfa0;  --ok-tint: #22352b;
  --warn: #dcaa54; --warn-tint: #3a2f1c;
}

/* ---------- 3. Expose to Tailwind (v4, CSS-first) ----------
   There is no tailwind.config.js — this block IS the config.
   Each --color-* line mints the utilities: bg-bg, text-fg,
   border-line, bg-accent, text-on-accent, … */
@theme inline {
  --color-bg: var(--bg);
  --color-fg: var(--fg);
  --color-surface: var(--surface);
  --color-line: var(--line);
  --color-muted: var(--muted);
  --color-accent: var(--accent);
  --color-accent-hover: var(--accent-hover);
  --color-accent-soft: var(--accent-soft);
  --color-tint: var(--tint);
  --color-on-accent: var(--on-accent);
  --color-ok: var(--ok);
  --color-ok-tint: var(--ok-tint);
  --color-warn: var(--warn);
  --color-warn-tint: var(--warn-tint);
  --font-display: "Fraunces Variable", Georgia, "Times New Roman", serif;
  --font-sans: "Public Sans Variable", "Helvetica Neue", Arial, sans-serif;
}

/* ---------- 4. Base ---------- */
body {
  background-color: var(--bg);
  color: var(--fg);
  font-family: var(--font-sans);
  -webkit-font-smoothing: antialiased;
}

/* Paper grain over everything (the "never flat white" rule);
   content sits above it via #root's z-index */
body::before {
  content: "";
  position: fixed;
  inset: 0;
  z-index: 0;
  pointer-events: none;
  opacity: 0.05;
  background-image: url("data:image/svg+xml,%3Csvg xmlns='http://www.w3.org/2000/svg' width='160' height='160'%3E%3Cfilter id='n'%3E%3CfeTurbulence type='fractalNoise' baseFrequency='0.85' numOctaves='2' stitchTiles='stitch'/%3E%3C/filter%3E%3Crect width='100%25' height='100%25' filter='url(%23n)'/%3E%3C/svg%3E");
}
#root { position: relative; z-index: 1; }

/* ---------- 5. Signature: the pairing thread ---------- */
/* The hero thread draws itself once on load (SVG stroke trick) */
.thread-draw {
  stroke-dasharray: 240;
  stroke-dashoffset: 240;
  animation: thread-draw 0.9s cubic-bezier(0.6, 0, 0.2, 1) 0.55s forwards;
}
.thread-dot { opacity: 0; animation: thread-pop 0.3s ease-out forwards; }
.thread-dot-a { animation-delay: 0.5s; }   /* pops before the line */
.thread-dot-b { animation-delay: 1.35s; }  /* pops after it lands */
@keyframes thread-draw { to { stroke-dashoffset: 0; } }
@keyframes thread-pop { to { opacity: 1; } }

/* Hero staggered rise — the ONE orchestrated motion moment */
.rise {
  opacity: 0;
  transform: translateY(14px);
  animation: rise 0.55s cubic-bezier(0.2, 0.7, 0.3, 1) forwards;
}
.rise-1 { animation-delay: 0.05s; }
.rise-2 { animation-delay: 0.15s; }
.rise-3 { animation-delay: 0.25s; }
.rise-4 { animation-delay: 0.4s; }
@keyframes rise { to { opacity: 1; transform: translateY(0); } }

/* Accessibility: ALL motion collapses to final states, in one place */
@media (prefers-reduced-motion: reduce) {
  .rise, .thread-dot { opacity: 1; transform: none; animation: none; }
  .thread-draw { stroke-dashoffset: 0; animation: none; }
  * { transition-duration: 0.01ms !important; }
}

/* ---------- 6. Overshoot heading underline ----------
   The negative left and +1.2em width push the bar past both edges
   of the heading — an ::after element, never a border. */
.overshoot { position: relative; display: inline-block; }
.overshoot::after {
  content: "";
  position: absolute;
  left: -0.15em;
  bottom: -0.18em;
  width: calc(100% + 1.2em);
  height: 3px;
  border-radius: 2px;
  background: var(--accent-soft);
}
```
