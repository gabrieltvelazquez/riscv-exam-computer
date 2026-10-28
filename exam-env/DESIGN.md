# Instructor console design

The console is used in a classroom, often on a laptop at the front of the
room, sometimes on a phone while walking between rows. It has one job: show
which exam boards are ready and let the instructor start or stop them with
confidence.

The visual system follows **Linear**. The upstream reference is vendored at
[`design/linear-reference.md`](design/linear-reference.md) (from
[awesome-design-md](https://github.com/VoltAgent/awesome-design-md)). This
file is the adaptation for the console and wins wherever the two disagree.
In short: a near-black canvas with a four-step charcoal surface ladder,
1px hairlines instead of shadows, Inter at 400-600 with gentle negative
tracking on headings, one lavender-blue accent for the primary action,
focus and the current nav item, and color otherwise spent only on status.

Principles, in priority order:

1. **State is never ambiguous.** Every board status has a color, a shape,
   and a word. Remove any one of them and the status is still readable.
2. **Destructive and fleet-wide actions are explicit.** The UI always says
   how many boards an action targets and which boards it skipped and why.
3. **Quiet by default, loud when it matters.** Hierarchy comes from the
   surface ladder and hairlines. Lavender marks the one thing to do next.
4. **Works offline.** No CDN fonts, scripts, or images. Inter and Geist Mono
   are vendored (OFL).

## Themes

Dark is the default, including under **Auto**, because the near-black
canvas is the Linear identity. **Light** is an explicit choice in the
sidebar toggle (`html[data-theme="light"]`).

## Tokens

All tokens are CSS custom properties on `:root` in `server/style/main.css`.

| Token             | Dark      | Light     | Use                                         |
| ----------------- | --------- | --------- | ------------------------------------------- |
| `--canvas`        | `#08090A` | `#F7F7F8` | Page, JSON preview, offline cards           |
| `--surface-1`     | `#0F1011` | `#FFFFFF` | Sidebar, cards, panels, tiles               |
| `--surface-2`     | `#141516` | `#F4F4F5` | Hovered cards, inputs, selection bar        |
| `--surface-3`     | `#1C1D1F` | `#EBEBED` | Current nav row, chips, actor tags          |
| `--hairline`      | `#23252A` | `#E4E4E7` | 1px card and divider lines                  |
| `--hairline-strong` | `#34343A` | `#D0D0D5` | Input borders, hovered cards             |
| `--ink`           | `#F7F8F8` | `#1A1A1E` | Primary text                                |
| `--ink-muted`     | `#D0D6E0` | `#3C4149` | Body text inside cards                      |
| `--ink-subtle`    | `#8A8F98` | `#6B6F76` | Labels, meta, nav                           |
| `--ink-tertiary`  | `#62666D` | `#9EA1A8` | Placeholders, zero counts                   |
| `--primary`       | `#5E6AD2` | `#5E6AD2` | Primary button, focus, selected card ring   |

Status colors (`--st-*`, each with a translucent `--st-*-wash`):

| Status     | Word      | Icon (shape)                   | Dark      | Light     |
| ---------- | --------- | ------------------------------ | --------- | --------- |
| `running`  | Running   | Solid disc with a slow pulse   | `#4CB782` | `#1F8A55` |
| `starting` | Starting… | Quarter-arc spinner            | `#F2C94C` | `#9A6B00` |
| `stopping` | Stopping… | Square inside a ring           | `#F2C94C` | `#9A6B00` |
| `idle`     | Ready     | Hollow ring                    | `#4EA7FC` | `#2569C3` |
| `error`    | Error     | Triangle with exclamation mark | `#EB5757` | `#CF3B3B` |
| `offline`  | Offline   | Dashed ring                    | `#8A8F98` | `#6B6F76` |

Badges are Linear-style: a neutral `--surface-2` pill with a hairline,
where only the status icon carries the color and the word stays in
`--ink-muted`. Summary tiles, card detail lines and the timeline use the
status color for text.

### Type

- **Inter** 400 / 500 / 600 (Linear's documented free substitute). Sentence
  case everywhere, no uppercase labels.
- **Geist Mono** 400 / 500 for hostnames, timestamps, JSON and stat values.

| Token     | Size / line height | Tracking | Use                        |
| --------- | ------------------ | -------- | -------------------------- |
| `--t-cap` | 12 / 16            | 0        | Labels, badges, meta       |
| `--t-mono`| 13 / 20            | 0        | Mono values                |
| `--t-sm`  | 14 / 21            | 0        | Body, controls (base)      |
| `--t-md`  | 15 / 22            | -0.1px   | Card and panel titles      |
| `--t-lg`  | 20 / 28            | -0.2px   | Section titles, sign-in    |
| `--t-xl`  | 28 / 34            | -0.6px   | Page titles                |

### Shape and depth

- Radius: 6px nav rows and small controls, 8px buttons and inputs, 12px
  cards and panels, 16px the sign-in card. Badges are full pills.
- **No shadows on resting surfaces.** Depth is the surface ladder plus a
  1px hairline. Only floating things (selection bar, sign-in card) get a
  dark drop shadow.
- Hover moves a card one step up the ladder; selected = lavender wash with
  a 1px lavender ring.
- Motion is 120ms color changes only.

## Layout

- Desktop: 232px `--surface-1` sidebar (college logo, nav, RISC-V mark,
  theme toggle, sign out). Content max width 1320px.
- < 960px: sidebar becomes a top bar with a horizontal nav row; the RISC-V
  mark and theme toggle are hidden there.
- Summary: seven separate compact tiles. Board grid
  `repeat(auto-fill, minmax(280px, 1fr))`.
- Exam downloads: ghost buttons ("Download .seb", "Download JSON"); "With password…" opens a full-width form row on `--canvas`; Delete sits at the far right.

## Brand marks

- `Logo`: college wordmark at the top of the sidebar and sign-in card;
  reversed white-text file on dark, blue-text file on light.
- `ProductMark`: small RISC-V Exam Computer mark (max 148px) in the sidebar
  footer and the foot of the sign-in card. On dark it uses a file with
  white "RISC" lettering instead of the navy one.

## Rules for future UI work

1. Use tokens. If a value is missing, add a token here first.
2. Never encode meaning in color alone. Pair it with an icon shape and a word.
3. Every action that changes boards reports what happened per board,
   including skipped boards and why.
4. Text from boards or instructors is untrusted. Render it as text.
5. Keep it offline: no external fonts, scripts, images, or analytics.
6. Check 360px width, keyboard-only use, reduced motion, and both themes.
7. Copy is plain and specific: "Start exam on 4 boards", not "Submit".
8. Mono for anything compared character by character.
9. Linear rules that apply here: one accent (lavender) only, no gradients,
   no decorative color, no pill buttons, no shadows on resting cards.
