# Instructor console design

The console is used in a classroom, often on a laptop at the front of the
room, sometimes on a phone while walking between rows. It has one job: show
which exam boards are ready and let the instructor start or stop them with
confidence.

The visual system follows **Vercel's Geist language**. The upstream
reference is vendored at [`design/vercel-reference.md`](design/vercel-reference.md)
(from [awesome-design-md](https://github.com/VoltAgent/awesome-design-md)).
This file is the adaptation for the console and wins wherever the two
disagree. In short: a stark ink-on-near-white duet (and its Geist dark
twin), white cards held by a 1px hairline ring and stacked soft shadows,
Geist for everything narrative, Geist Mono for anything technical, 6px
in-app radii, and color spent only on status. The brand mesh gradient
appears once, behind the sign-in card, the console's only hero surface.

Principles, in priority order:

1. **State is never ambiguous.** Every board status has a color, a shape,
   and a word. Remove any one of them and the status is still readable.
2. **Destructive and fleet-wide actions are explicit.** The UI always says
   how many boards an action targets and which boards it skipped and why.
3. **Quiet by default, loud when it matters.** Hairlines and spacing do the
   structure; color is spent on status and on the security warning only.
4. **Works offline.** No CDN fonts, scripts, or images. Everything is served
   by the exam server (Geist and Geist Mono are vendored, OFL).

## Tokens

All tokens are CSS custom properties on `:root` in `server/style/main.css`.
Components use tokens, never raw values.

### Themes

Light is the default. Dark (Geist dark) applies when the system prefers it,
or when the instructor picks it in the sidebar's Auto / Light / Dark toggle
(stored in the `theme` cookie, applied as `html[data-theme]`).

### Color

| Token              | Light     | Dark      | Use                                     |
| ------------------ | --------- | --------- | --------------------------------------- |
| `--canvas`         | `#FFFFFF` | `#0A0A0A` | Cards, panels, sidebar, inputs          |
| `--canvas-soft`    | `#FAFAFA` | `#000000` | Page background, card stat wells        |
| `--canvas-soft-2`  | `#F2F2F2` | `#1A1A1A` | Hover fills, current nav row, code bg   |
| `--ink`            | `#171717` | `#EDEDED` | Primary text                            |
| `--body`           | `#4D4D4D` | `#A1A1A1` | Secondary text                          |
| `--mute`           | `#8F8F8F` | `#8F8F8F` | Labels, hints, placeholders             |
| `--hairline`       | `#EBEBEB` | `#242424` | 1px dividers                            |
| `--hairline-strong`| `#C9C9C9` | `#454545` | Input borders, offline card outline     |
| `--primary`        | `#171717` | `#EDEDED` | Primary button fill, nav indicator      |
| `--on-primary`     | `#FFFFFF` | `#0A0A0A` | Text on primary                         |
| `--link` / `--focus` | `#0070F3` | `#52A8FF` | Focus ring, sparkline                 |
| `--code-bg`        | `#171717` | `#000000` | JSON preview (code-editor mockup)       |

### Status colors (semantic)

Each status has a foreground (`--st-*`) and an opaque wash (`--st-*-wash`).
**Status is always shown as icon + word + color.** Offline cards are also
sunk to `--canvas-soft` with a strong outline, so a grayscale screenshot
still reads correctly.

| Status     | Word         | Icon (shape)                    | Light     | Dark      |
| ---------- | ------------ | ------------------------------- | --------- | --------- |
| `running`  | Running      | Solid disc with a slow pulse    | `#297A3A` | `#62C073` |
| `starting` | Starting…    | Quarter-arc spinner             | `#AB570A` | `#FFB224` |
| `stopping` | Stopping…    | Square inside a ring            | `#AB570A` | `#FFB224` |
| `idle`     | Ready        | Hollow ring                     | `#0068D6` | `#52A8FF` |
| `error`    | Error        | Triangle with exclamation mark  | `#DA2F35` | `#FF6166` |
| `offline`  | Offline      | Dashed ring                     | `#7D7D7D` | `#8F8F8F` |

Starting and stopping share a hue on purpose, since both mean "in
transition, wait". Their shapes and words differ.

The security warning (no agent token) uses `--warn` on `--warn-wash` with a
`--warn-line` hairline.

### Type

- **Geist** 400 / 500 / 600 for all UI text. 600 is the ceiling; never 700.
- **Geist Mono** 400 / 500 for hostnames, IDs, JSON, timestamps, and the
  small uppercase labels (stat names, summary tile labels, actor chips,
  "Skipped"). Uppercase is allowed only in mono labels. Numbers always use
  `font-variant-numeric: tabular-nums`.
- Headings are sentence case with negative tracking.

| Token      | Size / line height | Tracking  | Use                                  |
| ---------- | ------------------ | --------- | ------------------------------------ |
| `--t-cap`  | 12 / 16            | 0         | Captions, hints, badges, mono labels |
| `--t-code` | 13 / 20            | 0         | JSON, stat values                    |
| `--t-sm`   | 14 / 20            | -0.28px   | Body, controls (the base size)       |
| `--t-md`   | 16 / 24            | -0.32px   | Card and panel titles, page subtitle |
| `--t-d-sm` | 20 / 28            | -0.6px    | Section titles (`h2`)                |
| `--t-d-md` | 24 / 32            | -0.96px   | Empty states, sign-in, phone `h1`    |
| `--t-d-lg` | 32 / 40            | -1.28px   | Page titles (`h1`), summary counts   |

### Space, radius, elevation, motion

- Spacing scale (4px base): `--s1` 4, `--s2` 8, `--s3` 12, `--s4` 16,
  `--s5` 24, `--s6` 32, `--s7` 48, `--s8` 64.
- Radius: `--r-xs` 4 (inline code), `--r-sm` 6 (buttons, inputs, nav rows,
  segmented control), `--r-md` 8 (banners, stat wells, JSON), `--r-lg` 12
  (cards, panels, selection bar, sign-in card), `--r-full` (badges, chips,
  live pill). **No pill buttons**: this is an app surface, so it stays on
  the 6px scale.
- Elevation is stacked, never one heavy drop:
  `--ring` (1px hairline) → `--shadow-2` (resting cards) → `--shadow-4`
  (hovered cards, result panel) → `--shadow-5` (selection bar, sign-in).
  Cards never scale or move on hover; only the shadow deepens.
- Motion: `.15-.2s ease` on color and shadow only. Status animations stop
  under `prefers-reduced-motion`.
- Hit targets are at least 40×40px; 44px on touch (`pointer: coarse`).

## Layout

- Desktop (≥ 960px): 248px white sidebar with a right hairline (wordmark,
  nav, theme toggle, sign out). The current nav row gets a `--canvas-soft-2`
  fill plus a 2px `--primary` indicator on its left edge. Content column
  max width 1400px with 48px gutters (32px below 1100px).
- Tablet / phone (< 960px): the sidebar becomes a top bar with the wordmark
  and a horizontal tab row; the current tab is underlined in `--primary`.
  Gutters drop to 16px. No horizontal page scroll at 360px width.
- Page head: `h1` + subtitle, closed by a hairline.
- Summary strip: one joined panel of tiles split by 1px hairlines.
- Board grid: `repeat(auto-fill, minmax(300px, 1fr))`; one column on phones.

## Components

| Component        | Notes                                                                          |
| ---------------- | ------------------------------------------------------------------------------ |
| `Logo`           | College wordmark image (`public/brand/`), blue-text on light, reversed white-text on dark. Above the "Exam Console" name in the sidebar and sign-in card; 54px wide in the top bar. |
| `ProductMark`    | Small RISC-V Exam Computer mark (max 156px) in the sidebar footer and at the foot of the sign-in card; dark themes use the white-lettering file (no plate); hidden in the phone top bar. |
| `Shell`          | Sidebar/topbar, nav with `aria-current="page"`, theme toggle, sign-out form.   |
| `Banner`         | Page-wide notice. Variant `warn` for the missing agent token.                   |
| `StatusBadge`    | Icon + word on its wash, full radius. `role="status"` is not used.             |
| `SummaryStrip`   | Mono label above a 32px count; tiles act as filters (`aria-pressed`), the pressed tile gets its status wash and a 2px bottom bar. |
| `BoardCard`      | Checkbox, name, mono hostname, badge, stat well (3 key stats), last seen, link. Selected = 2px ink ring. |
| `SelectionBar`   | Floating white bar (`--shadow-5`) when ≥ 1 board selected: count, exam picker, Start, Stop. |
| `ResultPanel`    | After an action: boards queued, boards skipped with the reason for each.       |
| `Button`         | 6px radius: `primary` (ink fill), `secondary` (canvas + ring), `danger` (red text + red ring; solid red when armed), `ghost`. |
| `Field`          | Label above, hint below, error text in `--st-error` with an icon. Inputs: canvas with a `--hairline-strong` inset border, focus = ink border + soft blue halo. |
| `Stat`           | Mono uppercase label + value; optional `Meter` (memory) or `Sparkline` (temp). |
| `Timeline`       | Activity rows split by hairlines: time (mono), actor chip (Instructor is ink-filled), text. |
| `ExamActions`    | Ghost buttons: "Download .seb", "Download JSON"; "With password…" opens a full-width form row on `--canvas-soft`; Delete sits at the far right. |
| `JsonPreview`    | Dark code-editor block (`--code-bg`), mono 13/20, scrolls inside itself.       |
| `EmptyState`     | Line illustration (inline SVG, `currentColor`), one sentence, one action.      |
| `ConfirmButton`  | Two-step destructive button: first click arms it ("Click again to forget").    |
| `ThemeToggle`    | Segmented Auto / Light / Dark control in the sidebar footer (hidden < 960px). |

## Rules for future UI work

1. Use tokens. If a value is missing, add a token here first.
2. Never encode meaning in color alone. Pair it with an icon shape and a word.
3. Every action that changes boards reports what happened per board,
   including skipped boards and why.
4. Text from boards or instructors (hostnames, names, URLs) is untrusted.
   Render it as text. Never use `inner_html` or build HTML strings.
5. Keep it offline: no external fonts, scripts, images, or analytics.
6. Check a 360px phone width, keyboard-only use (visible focus,
   logical tab order), `prefers-reduced-motion`, and **both themes**.
7. Copy is plain and specific: "Start exam on 4 boards", not "Submit".
   Use sentence case. No exclamation marks.
8. Mono font for anything a person might compare character by character:
   hostnames, IPs, hashes, versions, timestamps, JSON.
9. Vercel rules that apply here: no sixth accent color, no pill buttons, no
   single heavy drop shadow, no weight above 600, and the mesh gradient
   only at hero scale (the sign-in page).
