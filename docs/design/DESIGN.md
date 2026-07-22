# CYF Mentor Matching — Design Reference

The ground-truth design spec for the app, generated **from the running app
itself**. Every screen and state is captured in [`screens/`](screens/) —
desktop light + dark and mobile light — and every value below is the one the
code actually uses.

**Regenerating after UI changes** (app running, from the repo root):

```bash
npm run design:stage      # idempotent: demo accounts frozen in each state
npm run design:capture    # re-captures all ~58 screenshots
```

A Penpot companion file exists with the token variables, text styles, and core
components: [CYF Mentor Matching — Design](https://design.penpot.app/#/workspace?team-id=a1a9e568-e174-80fb-8008-5d048cb5e258&project-id=7f4897ee-b258-8011-8008-5d7f7a500505&file-id=9c0d2cc0-f14a-8107-8008-5d7fa6d2f0fe&page-id=9c0d2cc0-f14a-8107-8008-5d7fa6d31670).

---

## 1. Design language: "The Pairing Thread"

One idea drives everything: **mentorship is two dots joined by a line**. The
thread appears in the wordmark, draws itself across the landing hero, connects
the two people on every match card, and anchors every empty state. Around it:
warm paper (never flat white), one red voice, and a serif that feels human.

---

## 2. Color tokens

Semantic tokens, defined once in `frontend/src/index.css` and consumed as
Tailwind classes (`bg-bg`, `text-fg`, `border-line`, …). The viewer's theme is
`prefers-color-scheme` plus a manual toggle that sets `data-theme` on `<html>`.

| Token | Light | Dark | Used for |
|---|---|---|---|
| `bg` | `#faf6f2` | `#191412` | page background (warm paper / warm charcoal) |
| `fg` | `#231d1b` | `#f2eae4` | body text (warm ink) |
| `surface` | `#ffffff` | `#251e1a` | cards, inputs, header fill |
| `line` | `#e9ddd5` | `#3d322c` | borders, dividers, off-state track |
| `muted` | `#6f6259` | `#a99a90` | secondary text |
| `accent` | `#d80f0f` | `#f5807f` | interactive: buttons, links, active nav — **AA-safe on bg/surface** |
| `accent-hover` | `#a71000` | `#ffa19f` | hover state of `accent` |
| `accent-soft` | `#ea4549` | `#ea4549` | decoration ONLY: thread, underlines, big numerals — fails AA for small text |
| `tint` | `#fbeae7` | `#38221f` | accent-tinted fills: active nav pill, selected role card |
| `on-accent` | `#ffffff` | `#1c1210` | text on `accent` fills (never use raw white/black) |
| `ok` | `#1f7a4d` | `#79cfa0` | success text/rings |
| `ok-tint` | `#e7f3ec` | `#22352b` | success pill fill |
| `warn` | `#875c15` | `#dcaa54` | warning text (AA on warn-tint) |
| `warn-tint` | `#f8efdd` | `#3a2f1c` | warning pill/banner fill |

**Rules:** `accent` is the only interactive color. `accent-soft` never carries
small text. Anything on an `accent` fill uses `on-accent` — that token is what
keeps buttons legible in every OS-theme × app-theme combination. Semantic
colors (`ok`/`warn`) are states, not decoration.

**Atmosphere:** the page body carries a paper-grain overlay (inline SVG
`feTurbulence` noise at 5% opacity, fixed, behind content) and the landing
hero has one soft radial glow (`accent-soft` at ~22%, blurred) anchored
top-right. Never flat.

---

## 3. Typography

Two families, self-hosted via `@fontsource` (no CDN):

- **Fraunces** (SemiBold) — display: every heading, stat numeral, person name.
- **Public Sans** — body: everything else. Regular for prose, SemiBold for
  labels, buttons, links.

| Role | Face | Size/line | Tracking | Where |
|---|---|---|---|---|
| Hero | Fraunces SB | 56–60 / 1.05 | −2% | landing h1 |
| H1 | Fraunces SB | 36 / 1.15 | −1.5% | page titles |
| H2 | Fraunces SB | 30 / 1.2 | −1% | section heads |
| Card title | Fraunces SB | 20 / 1.3 | 0 | person names, empty-state lines |
| Body | Public Sans R | 17 / 1.65 | 0 | prose |
| Small | Public Sans R | 14 / 1.5 | 0 | table cells, sublabels |
| Small SB | Public Sans SB | 14 / 1.4 | 0 | buttons, links, nav |
| Eyebrow | Public Sans SB | 12 / 1.3 | +9–14%, uppercase | landing eyebrow, card role labels |

**Signature heading treatment:** H1/H2 sit on an "overshoot" underline — a
3px `accent-soft` bar, radius 2, extending ~0.15em past the left edge and
~1.2em past the right of the text (`.overshoot` in `index.css`).

---

## 4. Component anatomy

Values are light-theme; swap tokens for dark automatically.

**Button** — radius 6, Public Sans SB 14, min-height 44px, padding 10×16
(hero CTAs 12×24). Primary: `accent` fill, `on-accent` text, hover
`accent-hover`. Outline: 1.5px `accent` border, `accent` text, hover `tint`
fill. Quiet: `muted` text, hover `tint`. Focus (all interactives): 2px
`accent` outline, 2px offset — never removed.

**Status pill** — radius full, padding 4×10, Public Sans SB 12. Proposed:
`tint`/`accent` "Awaiting acceptance". Accepted: `warn-tint`/`warn`
"Chemistry & confirm". Active: `ok-tint`/`ok` "Active". Closed: `line` at
60%/`muted`.

**Thread (signature)** — 10px dot, 2.5px line (slight S-curve in SVG), 10px
dot, all `accent-soft`. Dots are real round elements; only the line
stretches. On the landing hero it draws itself in 0.9s (stroke-dashoffset),
dots popping in before/after.

**Match card** — `surface` fill, 1px `line` border, radius 10, padding 24.
Rows: PairRow (name right-aligned | thread + status pill, 128–144px wide |
name left-aligned; names Fraunces SB 20, sublabels 12 `muted`), then contact
block ("REACH {NAME}" eyebrow + underlined `accent` links), then action row
split by a `line` divider: countdown/status left, buttons right.

**Journey rail (mentee)** — 4 steps: 14px dot + 14px label + 48px connector.
Done: filled `accent-soft` dot, `muted` struck-through label, `accent-soft`
connector. Current: 2px `accent-soft` ring, SB `fg` label. Todo: 2px `line`
ring, `muted`.

**Chips (disciplines/goals/availability)** — radius full, padding 9×16, SB
14, min-height 44. On: `accent` fill + `on-accent` text. Off: `surface` +
1.5px `line` border, hover border `accent`. They are toggle buttons
(`aria-pressed`), not checkboxes.

**Switch** — track 44×24 radius-full (`accent` on / `line` off), knob 20px
white, 2px inset, translates 20px. `role="switch"` + `aria-checked`.

**Inputs** — `surface` fill, 1.5px `line` border, radius 6, padding 10×14,
15px text; focus: border `accent` + outline. Labels above, SB 14.

**Stat tile (staff)** — card + Fraunces SB 40 numeral (`fg`, or `ok`/`warn`
when the number is itself a status), 14px `muted` label under.

**Capacity ring (mentor)** — 72px SVG ring, 7px stroke: `ok` under capacity,
`accent-soft` at capacity, `line` track. Fraunces "live/capacity" centered.

**Tabs (staff)** — text SB 14 on a `line` bottom border; active: `accent`
text + 2px `accent-soft` underline sitting on the border.

**Notices** — 4px left border, radius 6, padding 12×16, `surface` fill.
Status: `accent-soft` border. Warning ("can't be matched yet"): `warn`
border + `warn-tint` fill.

**App header** — sticky, `bg` at 90% + backdrop-blur, 1px `line` bottom
border, 64px tall, max-width 1152 (max-w-6xl) centered — the content column
everywhere. Mobile (< 768px): nav collapses to a hamburger opening a 288px
right slide-over drawer (`surface`, left `line` border) with links, user
info and logout pinned at the bottom; Escape and backdrop close it.

---

## 5. Layout & motion

- Content column: max-width 1152px, 16px side padding; forms max-width 384px.
- Section spacing: 40px between page sections; 96px paddings on landing bands.
- Landing hero is asymmetric: copy column ~1.1fr, pair-visual ~1fr, the two
  hero cards rotated ±2°.
- Motion: ONE orchestrated moment — landing hero lines rise staggered
  (0.55s, 50–400ms delays) and the thread draws itself. Everything else is
  color transitions. `prefers-reduced-motion` disables all of it.
- Wide content (staff tables) scrolls inside its own container, never the page.

---

## 6. Screenshot index

Files follow `{screen}--{state}--{variant}.png`; variants are
`desktop-light`, `desktop-dark`, `mobile-light`.

| Screen / state | Route | Logged in as |
|---|---|---|
| `landing` | `/` | — |
| `login` | `/login` | — |
| `signup` | `/signup` (role picker) | — |
| `mentee-profile--incomplete` | `/profile` — warn banner | stage.incomplete@ |
| `mentee-dashboard--incomplete` | `/dashboard` — gated CTA | stage.incomplete@ |
| `mentee-dashboard--proposed` | Accept/Decline, countdown | stage.proposed@ |
| `mentee-dashboard--accepted` | contact block + "I've booked our session" | stage.accepted@ |
| `mentee-dashboard--booked` | booked ✓ + "Confirm mentorship" | stage.booked@ |
| `mentee-dashboard--active` | active mentorship | demo.mentee@ |
| `mentee-profile--complete` | full editor + questionnaire | stage.booked@ |
| `mentee-dashboard--ready-history` | matchable + enabled CTA + past matches | stage.history@ |
| `mentee-dashboard--confirmed-waiting` | mentee not confirmed, mentor has | stage.confirmed@ |
| `signup--mentor-role` | mentor role card selected | — |
| `login--error` | invalid-credentials alert | — |
| `mentor-profile--incomplete` | warn banner (disciplines missing) | stage.mentor.new@ |
| `mentor-profile--complete` | matchable + availability nudge | ruta.radiya@ |
| `mentor-dashboard--empty` | no mentees, **paused** variant | gate.mentor@ |
| `mentor-dashboard--empty-accepting` | no mentees, accepting variant | stage.mentor.open@ |
| `mentor-dashboard--proposed-wait` | "waiting for X to accept" | staged-accounts.json |
| `mentor-dashboard--awaiting-booking` | "waiting for the mentee to book" | staged-accounts.json |
| `mentor-dashboard--confirm` | booked mentee, Confirm button | staged-accounts.json |
| `mentor-dashboard--confirmed-waiting` | mentor confirmed, mentee hasn't | staged-accounts.json |
| `mentor-dashboard--active` | active mentee, End mentorship | ruta.radiya@ |
| `staff-overview` | stat tiles + waiting queue | staff@ |
| `staff-mentors` | search + mentors table | staff@ |
| `staff-mentors--search-empty` | empty-search table row | staff@ |
| `staff-mentees` | search + mentees table | staff@ |
| `staff-settings` | disciplines + questions management | staff@ |
| `staff-settings--add-question` | new-question form open | staff@ |
| `staff-user-detail` | mentor detail: facts + audit trail | staff@ |
| `staff-user-detail--mentee` | mentee detail: propose-pair card | staff@ |
| `mobile-drawer--open` | mobile nav drawer | stage.booked@ |

Mentor-side logins for match states depend on who the matcher picked — they
are resolved and claimed at staging time and recorded in
`staged-accounts.json`. All staged accounts use the demo password from the
README.

**States deliberately not captured** (transient or destructive to stage):
save-confirmation toasts ("Profile saved", "Goals saved"), the staff
propose-match result notices, and the mentee "no compatible mentor" notice —
the last would require draining all 60+ mentors' capacity to trigger. Their
copy lives in the source (`MenteeDashboard.tsx`, `StaffDashboard.tsx`,
`Profile.tsx`). Note also that `stage.history` re-enters the matching queue,
so the hourly sweep may re-match them — re-run `design:stage` before
`design:capture` for pristine states.

---

## 7. What screenshots can't show

- **Hover:** buttons darken to `accent-hover`; outline buttons and chips gain
  `tint` fill / `accent` border; links underline-shift to `accent-hover`.
- **Focus:** every interactive element shows the 2px offset `accent` outline.
- **The hero animation** (§5) — see the running app.
- **Reduced motion:** all animation collapses to instant final states.
