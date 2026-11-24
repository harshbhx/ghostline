# Ghostline Design System

> Voice agents that know when to shut up.

| | |
|---|---|
| Repo path | `docs/DESIGN.md` |
| Applies to | Docs site (**ghostline.dualmindlab.tech**), Missed-Call Oracle, README, social images (parent) · Dead Air FM (child) |
| Source of truth | `design/tokens.css` (CSS variables) and `design/tokens.json`. This file explains them. If they disagree, the token files win; then fix this file. |
| Visual reference | Brand board canvas (Brand, Docs site and Dead Air FM pages) and the "Ghostline" design system artifact |
| Status | v3 · 20 Nov 2026 · Graphite + Blue parent, Dead Air FM child |

## 0. Rules for AI agents

1. **Never write a raw hex colour, font name or pixel spacing in components.** Use the CSS variables from `design/tokens.css` (`var(--text)`, `var(--space-4)`, `var(--radius-md)`). The only exception is `design/tokens.css` itself.
2. **Use semantic tokens, not ramp tokens.** `var(--accent)`, not `var(--blue-400)`. Ramps exist so semantic tokens can point at them.
3. **Know which family member you are building.** The docs site, Missed-Call Oracle and everything else use the **parent** (`data-theme="dark"` or `"light"`). Only Dead Air FM uses the **child** (`data-theme="deadair"`). Never use `night-*`, `valve-*`, the serif or the effects outside Dead Air FM.
4. **Colour has one meaning each.** Blue = listening, links, "go". Orange = you cut in, live, error. Amber = warning. Never decoration.
5. **Numbers are honest.** Show `—` for anything not measured. Never invent a value, never show `0` as a placeholder.
6. If something is not covered here, ask. Do not invent new colours, fonts or sizes.

---

## 1. One family: parent and child

```
Ghostline (parent) ─ Graphite + Blue, Geist, Night and Day
├── Docs site · ghostline.dualmindlab.tech        uses the parent as is
├── Missed-Call Oracle (browser simulator)         uses the parent as is
└── Dead Air FM (child) ─ data-theme="deadair"     inherits Night, overrides a short list
```

The child **inherits every token from Night** and overrides only the colours in the "Dead Air FM" column of 3.2. A dash in that column means "same as Night". In CSS this is literal: the `[data-theme="deadair"]` block in `tokens.css` declares only the overrides, and everything else falls through from `:root`.

**The child may change:** surface colours (blue-black night instead of graphite), text colour (bone instead of grey-white), the speaking colour (a warm valve glow), the primary button (a bone pill), the display face for its own big moments (Instrument Serif), and it may add atmosphere (grain, scanlines, vignette, fog, a moon glow).

**The child may not change:** blue for listening, orange for interruption, Geist for every piece of UI text, Geist Mono for numbers, the spacing scale, the radius scale, the logo shape, the honesty rule, or any accessibility rule. That shared core is what makes Dead Air FM read as a Ghostline app at a glance.

---

## 2. Principles

1. **Neutral first, colour only where it means something.** The parent is almost entirely greys. Blue and orange are reserved for state, so they carry information every time they appear.
2. **Borders, not shadows.** Surfaces are separated by 1 px hairlines (`--line`). Only floating layers (search dialog, menus) get `--shadow-overlay`.
3. **Quiet by default, loud on events.** Things move only when state changes. The biggest motion in the parent is the logo in the home hero.
4. **Readable first.** Docs prose is 16/27 Geist at 68 characters per line. Nothing decorative may reduce readability.
5. **The child earns its mood from atmosphere, not props.** Dead Air FM is a pirate radio station at 3 a.m.: grain, fog, moonlight, a valve glow. No pumpkins, bats, skulls, cobwebs or purple.

---

## 3. Colour

### 3.1 Ramps

Higher number = darker. `night` and `valve` belong to Dead Air FM only.

| Step | Gray | Blue | Orange | Amber | Night | Valve |
|---|---|---|---|---|---|---|
| 0 | `#ffffff` |  |  |  | `#ece6da` |  |
| 50 | `#fafafa` | `#eef4ff` | `#fff1eb` | `#fdf4e1` |  |  |
| 100 | `#f2f2f2` | `#dbe7ff` | `#ffe0d1` |  |  |  |
| 150 | `#ededed` |  |  |  |  |  |
| 200 | `#e8e8e8` | `#b9d1ff` |  |  |  |  |
| 300 | `#d1d1d1` | `#8ab8ff` | `#ffb08f` | `#f3cc7c` | `#958f86` | `#ffd29c` |
| 400 | `#a1a1a1` | `#5b9bff` | `#ff6a33` | `#e8b04a` |  | `#ffb257` |
| 500 | `#8f8f8f` | `#2f6fef` |  |  |  | `#ff5a2e` |
| 550 | `#737373` |  |  |  |  |  |
| 600 | `#5e5e5e` | `#1f5fd6` | `#c8400f` |  | `#45423e` |  |
| 700 | `#3a3a3a` | `#174bb0` | `#a5340b` | `#8a5a00` |  |  |
| 750 | `#262626` |  |  |  | `#2a2826` |  |
| 800 | `#1a1a1a` | `#133c8a` | `#8a2a07` | `#6b4500` | `#1d1c1b` |  |
| 850 | `#111111` |  |  |  | `#131418` |  |
| 900 | `#0a0a0a` | `#0f2b5e` | `#4a1a08` |  | `#0c0d10` |  |
| 950 |  | `#0c1830` | `#2a140c` | `#271d08` | `#06070a` |  |

### 3.2 Semantic tokens

A dash in the Dead Air FM column means the child inherits Night's value.

| Token | Night | Day | Dead Air FM | Use |
|---|---|---|---|---|
| `--bg` | `gray-900` `#0a0a0a` | `gray-0` `#ffffff` | `night-950` `#06070a` | Page background. |
| `--surface` | `gray-850` `#111111` | `gray-50` `#fafafa` | `night-900` `#0c0d10` | Cards, sidebar, header, panels. |
| `--surface-raised` | `gray-800` `#1a1a1a` | `gray-100` `#f2f2f2` | `night-850` `#131418` | Hover rows, active nav item, inputs, kbd. |
| `--surface-overlay` | `gray-800` `#1a1a1a` | `gray-0` `#ffffff` | `night-850` `#131418` | Menus, search dialog, popovers (with shadow-overlay). |
| `--line` | `gray-750` `#262626` | `gray-200` `#e8e8e8` | `night-800` `#1d1c1b` | Hairlines, dividers, table rules, card borders. |
| `--line-strong` | `gray-700` `#3a3a3a` | `gray-300` `#d1d1d1` | `night-750` `#2a2826` | Secondary button borders and strong dividers. Decorative: never the only cue. |
| `--line-control` | `gray-550` `#737373` | `gray-500` `#8f8f8f` | — | Borders that must be seen: inputs, checkboxes, toggles (3:1 on bg). |
| `--text` | `gray-150` `#ededed` | `gray-900` `#0a0a0a` | `night-0` `#ece6da` | Body and headings. |
| `--text-muted` | `gray-400` `#a1a1a1` | `gray-600` `#5e5e5e` | `night-300` `#958f86` | Secondary text, captions, labels (4.5:1 or more). |
| `--text-faint` | `gray-550` `#737373` | `gray-500` `#8f8f8f` | `night-600` `#45423e` | Disabled text, decorative numbers, words not yet spoken. Never for content you must read. |
| `--accent` | `blue-400` `#5b9bff` | `blue-600` `#1f5fd6` | — | Links, active nav, focus, selected items. Text-safe on bg and surface. |
| `--accent-hover` | `blue-300` `#8ab8ff` | `blue-700` `#174bb0` | — | Hover on links. |
| `--accent-fill` | `blue-500` `#2f6fef` | `blue-600` `#1f5fd6` | `night-0` `#ece6da` | Primary button fill. Dead Air FM uses a bone-white pill instead. |
| `--accent-fill-hover` | `blue-600` `#1f5fd6` | `blue-700` `#174bb0` | `gray-0` `#ffffff` | Primary button hover. |
| `--on-accent` | `gray-0` `#ffffff` | `gray-0` `#ffffff` | `night-950` `#06070a` | Text and icons on accent-fill. |
| `--accent-soft` | `blue-950` `#0c1830` | `blue-50` `#eef4ff` | — | Tinted background for a selected row or the caller's bubble. |
| `--accent-soft-text` | `blue-300` `#8ab8ff` | `blue-700` `#174bb0` | — | Text inside accent-soft. |
| `--alarm` | `orange-400` `#ff6a33` | `orange-600` `#c8400f` | `valve-500` `#ff5a2e` | Barge-in, errors, destructive actions. |
| `--alarm-soft` | `orange-950` `#2a140c` | `orange-50` `#fff1eb` | — | Tinted background for an interruption or error. |
| `--alarm-soft-text` | `orange-300` `#ffb08f` | `orange-800` `#8a2a07` | — | Text inside alarm-soft. |
| `--amber` | `amber-400` `#e8b04a` | `amber-700` `#8a5a00` | — | Warnings. |
| `--amber-soft` | `amber-950` `#271d08` | `amber-50` `#fdf4e1` | — | Tinted background for a warning. |
| `--amber-soft-text` | `amber-300` `#f3cc7c` | `amber-800` `#6b4500` | — | Text inside amber-soft. |
| `--mark-eye` | `blue-400` `#5b9bff` | `blue-600` `#1f5fd6` | `valve-400` `#ffb257` | The ghost's lamp eye at rest, and the wordmark's i-dot. |
| `--on-air` | `orange-400` `#ff6a33` | `orange-600` `#c8400f` | `valve-400` `#ffb257` | Live / ON AIR indicator. |
| `--glow` | `#5b9bff1f` | `#1f5fd60f` | `#ffb25740` | Soft light behind the mark in a hero panel. Never behind text. |
| `--focus` | `blue-400` `#5b9bff` | `blue-600` `#1f5fd6` | — | Focus ring: 2px solid, 2px offset. |
| `--selection` | `blue-900` `#0f2b5e` | `blue-100` `#dbe7ff` | — | Text selection background. |

### 3.3 State colours

The four lamp states. Listening is blue and an interruption is orange in **every** theme. Blue/orange stays distinct for the common forms of colour blindness, and every lamp also has a text label.

| Token | Night | Day | Dead Air FM | Use |
|---|---|---|---|---|
| `--state-speaking` | `gray-150` `#ededed` | `gray-900` `#0a0a0a` | `valve-400` `#ffb257` | Lamp: the bot is talking. In Dead Air FM the host glows like a valve. |
| `--state-thinking` | `gray-400` `#a1a1a1` | `gray-600` `#5e5e5e` | `night-300` `#958f86` | Lamp: waiting for the model. |
| `--state-heard` | `orange-400` `#ff6a33` | `orange-600` `#c8400f` | `valve-500` `#ff5a2e` | Lamp: you cut in, barge-in confirmed. |
| `--state-listening` | `blue-400` `#5b9bff` | `blue-600` `#1f5fd6` | — | Lamp: listening to you. Blue in every theme. |

### 3.4 Code colours

Code is always on the dark console, in every theme.

| Token | Night | Day | Dead Air FM | Use |
|---|---|---|---|---|
| `--code-bg` | `gray-850` `#111111` | `gray-850` `#111111` | `night-900` `#0c0d10` | Code blocks. Code is always on the dark console, in every theme. |
| `--code-line` | `gray-750` `#262626` | `gray-750` `#262626` | `night-800` `#1d1c1b` | Code block border and tab bar rule. |
| `--syn-text` | `gray-150` `#ededed` | `gray-150` `#ededed` | `night-0` `#ece6da` | Code: plain text. |
| `--syn-comment` | `gray-500` `#8f8f8f` | `gray-500` `#8f8f8f` | `night-300` `#958f86` | Code: comments, italic. |
| `--syn-keyword` | `blue-300` `#8ab8ff` | `blue-300` `#8ab8ff` | — | Code: keywords. |
| `--syn-type` | `blue-200` `#b9d1ff` | `blue-200` `#b9d1ff` | — | Code: types and classes. |
| `--syn-string` | `amber-300` `#f3cc7c` | `amber-300` `#f3cc7c` | — | Code: strings. |
| `--syn-number` | `orange-300` `#ffb08f` | `orange-300` `#ffb08f` | — | Code: numbers and constants. |
| `--syn-property` | `gray-400` `#a1a1a1` | `gray-400` `#a1a1a1` | — | Code: properties, attributes, punctuation. |

### 3.5 Measured contrast

Text needs 4.5 : 1. Lamps, focus rings and control borders need 3 : 1. All pairs below pass in all three themes; `tokens.py` fails the build if one stops passing.

| Pair | Needs | Night | Day | Dead Air FM |
|---|---|---|---|---|
| `text` on `bg` | 4.5 : 1 | 16.91 | 19.80 | 16.21 |
| `text` on `surface` | 4.5 : 1 | 16.13 | 18.97 | 15.64 |
| `text` on `surface-raised` | 4.5 : 1 | 14.87 | 17.68 | 14.81 |
| `text-muted` on `bg` | 4.5 : 1 | 7.66 | 6.48 | 6.28 |
| `text-muted` on `surface` | 4.5 : 1 | 7.31 | 6.21 | 6.06 |
| `text-muted` on `surface-raised` | 4.5 : 1 | 6.74 | 5.79 | 5.74 |
| `accent` on `bg` | 4.5 : 1 | 7.14 | 5.73 | 7.27 |
| `accent` on `surface` | 4.5 : 1 | 6.81 | 5.49 | 7.01 |
| `on-accent` on `accent-fill` | 4.5 : 1 | 4.52 | 5.73 | 16.21 |
| `alarm` on `bg` | 4.5 : 1 | 6.94 | 5.00 | 6.48 |
| `alarm` on `surface` | 4.5 : 1 | 6.62 | 4.79 | 6.25 |
| `amber` on `bg` | 4.5 : 1 | 10.13 | 5.93 | 10.30 |
| `accent-soft-text` on `accent-soft` | 4.5 : 1 | 8.73 | 7.12 | 8.73 |
| `alarm-soft-text` on `alarm-soft` | 4.5 : 1 | 9.85 | 7.88 | 9.85 |
| `amber-soft-text` on `amber-soft` | 4.5 : 1 | 10.86 | 7.75 | 10.86 |
| `state-listening` on `surface` | 3 : 1 | 6.81 | 5.49 | 7.01 |
| `state-heard` on `surface` | 3 : 1 | 6.62 | 4.79 | 6.25 |
| `state-speaking` on `surface` | 3 : 1 | 16.13 | 18.97 | 10.88 |
| `line-control` on `bg` | 3 : 1 | 4.18 | 3.23 | 4.25 |
| `focus` on `bg` | 3 : 1 | 7.14 | 5.73 | 7.27 |
| `syn-text` on `code-bg` | 4.5 : 1 | 16.13 | 16.13 | 15.64 |
| `syn-comment` on `code-bg` | 4.5 : 1 | 5.84 | 5.84 | 6.06 |
| `syn-keyword` on `code-bg` | 4.5 : 1 | 9.33 | 9.33 | 9.61 |
| `syn-type` on `code-bg` | 4.5 : 1 | 12.25 | 12.25 | 12.61 |
| `syn-string` on `code-bg` | 4.5 : 1 | 12.35 | 12.35 | 12.71 |
| `syn-number` on `code-bg` | 4.5 : 1 | 10.66 | 10.66 | 10.97 |
| `syn-property` on `code-bg` | 4.5 : 1 | 7.31 | 7.31 | 7.52 |

### 3.6 Colour rules

- One accent per screen: blue. Orange appears only when something is live or you interrupted.
- Primary buttons use `--accent-fill` with `--on-accent`. On dark, the fill is one step deeper than link blue so white text passes 4.5 : 1.
- `--line-strong` is decorative (secondary button borders). Anything a user must find (inputs, checkboxes, toggles) uses `--line-control`.
- `--text-faint` is never used for text a user needs. Dead Air FM uses it for words the host has not spoken yet.
- `--glow` is the only translucent colour. It sits behind the mark in a hero panel, never behind text.

---

## 4. Typography

### 4.1 Families

| Token | Stack | Used for |
|---|---|---|
| `--font-display` | Geist | Headings. Same family as text, at 600 with tight tracking. |
| `--font-sans` | Geist | Everything else in the UI. |
| `--font-mono` | Geist Mono | Code, numbers, keyboard keys. |
| `--font-serif` | Instrument Serif | **Dead Air FM only:** the station name, the host's live words, card titles. |

All four are on Google Fonts (Geist and Geist Mono are also on npm as `geist`). Self-host in production. The wordmark in the logo is outlined artwork, not live text.

### 4.2 Parent scale

| Style | Family | Size / line | Weight | Tracking | Use |
|---|---|---|---|---|---|
| `.t-display-xl` | display | 72px / 72px | 600 | -0.035em | Docs home hero only. |
| `.t-display-l` | display | 48px / 52px | 600 | -0.03em | Section openers on the home page. |
| `.t-h1` | display | 40px / 46px | 600 | -0.025em | Page title, once per page. |
| `.t-h2` | display | 28px / 34px | 600 | -0.02em | Section heading in docs. |
| `.t-h3` | sans | 20px / 28px | 600 | -0.01em | Sub-section heading, card title. |
| `.t-h4` | sans | 16px / 24px | 600 | 0em | Small heading. |
| `.t-body-l` | sans | 18px / 30px | 400 | 0em | Lead paragraph under a page title. |
| `.t-body` | sans | 16px / 27px | 400 | 0em | Docs prose. Max line length 68ch. |
| `.t-body-s` | sans | 14px / 22px | 400 | 0em | Sidebar, tables, UI text. |
| `.t-caption` | sans | 13px / 20px | 400 | 0em | Captions, footnotes, helper text. |
| `.t-label` | sans | 13px / 18px | 500 | 0em | Eyebrows, table headers, lamp labels. Sentence case. |
| `.t-code` | mono | 14px / 24px | 400 | 0em | Code blocks. Inline code uses 0.875em. |
| `.t-data-xl` | mono | 44px / 48px | 500 | -0.02em | Big measured numbers. Tabular figures. |
| `.t-data` | mono | 16px / 24px | 500 | 0em | Numbers in tables and meters. Tabular figures. |

### 4.3 Dead Air FM additions

| Style | Family | Size / line | Weight | Tracking | Use |
|---|---|---|---|---|---|
| `.t-station` | serif | 168px / 151px | 400 | -0.03em | Dead Air FM only: the station name on the tune-in screen. |
| `.t-transcript` | serif | 40px / 52px | 400 | -0.01em | Dead Air FM only: the host's live words, desktop. |
| `.t-transcript-s` | serif | 27px / 35px | 400 | 0em | Dead Air FM only: the host's live words, mobile; card titles. |
| `.t-dial` | mono | 12px / 16px | 500 | 0.06em | Dead Air FM only: uppercase readouts (ON AIR, frequencies, clock). |

Effects (Dead Air FM only):

| Token | Value | Use |
|---|---|---|
| `--fx-grain` | 0.07 | Dead Air FM only: opacity of the moving film grain layer. |
| `--fx-scanline` | 0.18 | Dead Air FM only: opacity of the 1px-in-3px scanline layer. |
| `--fx-vignette` | 0.65 | Dead Air FM only: edge darkness of the vignette. |

### 4.4 Rules

- Headings are Geist 600 with negative tracking; body is Geist 400. No other weights except 500 for labels and data.
- Labels are **sentence case**, Geist 500 13 px. The parent has no uppercase text. (Dead Air FM's `dial` style is the one uppercase style, for readouts like ON AIR and frequencies.)
- Numbers use tabular figures (`font-variant-numeric: tabular-nums`).
- `text-wrap: balance` on headings, `pretty` on leads.

---

## 5. Space, shape, layout

### 5.1 Spacing (4 px base)

| Token | Value | Use |
|---|---|---|
| `--space-0_5` | 2px | Hairline gaps, icon nudges. |
| `--space-1` | 4px | Base unit. Gap between a lamp and its label. |
| `--space-2` | 8px | Tight gaps: chips, inline groups. |
| `--space-3` | 12px | Nav item padding, small card gap. |
| `--space-4` | 16px | Default gap, card padding on mobile. |
| `--space-5` | 20px | Code block padding. |
| `--space-6` | 24px | Card padding, grid gap. |
| `--space-8` | 32px | Between a section's parts. |
| `--space-10` | 40px | Page side padding on desktop. |
| `--space-12` | 48px | Before an h2. |
| `--space-16` | 64px | Between home sections on mobile. |
| `--space-24` | 96px | Between home sections on desktop. |

### 5.2 Radius

| Token | Value | Use |
|---|---|---|
| `--radius-xs` | 2px | Kbd keys, tiny tags. |
| `--radius-sm` | 4px | Inline code, chips. |
| `--radius-md` | 8px | Buttons, inputs, search. |
| `--radius-lg` | 12px | Cards, code blocks, callouts, dialogs. |
| `--radius-xl` | 16px | Hero panels, app icon tiles, large media. |
| `--radius-pill` | 999px | Lamps, status pills, Dead Air FM buttons. |

### 5.3 Shadow

Borders, not shadows. `--shadow-overlay` is for search, menus and popovers only. Dead Air FM inherits Night's shadow.

### 5.4 Layout sizes

| Token | Value | Use |
|---|---|---|
| `--size-header` | 64px | Docs header height. |
| `--size-sidebar` | 264px | Left nav width. |
| `--size-toc` | 224px | On-this-page column width. |
| `--size-prose` | 720px | Max width of doc content. |
| `--size-page` | 1360px | Max width of the whole layout. |
| `--size-hit` | 44px | Minimum touch target. |

### 5.5 Breakpoints

| Width | Docs layout |
|---|---|
| ≥ 1280 px | Header + sidebar (264) + content (≤ 720) + on-this-page (224), all inside 1360 px, centred |
| 1024–1279 px | On-this-page column hides; content grows |
| 768–1023 px | Sidebar becomes a drawer opened by a menu button in the header |
| < 768 px | Single column; header search becomes an icon button; page padding `--space-4` |

Touch targets are at least `--size-hit` (44 px) everywhere.

---

## 6. Motion

| What | Duration | Easing |
|---|---|---|
| Lamp colour change | 120 ms | ease-out |
| Barge-in flash | at most 150 ms | linear |
| Menu, dialog open | 160 ms fade + 4 px rise | cubic-bezier(.16, 1, .3, 1) |
| Hover colour | 120 ms | ease-out |
| Theme switch | none | — |
| Logo loop (home hero) | 5.2 s loop | defined in the logo files |

- Parent: the animated logo loops only in the home hero. In the header it is static.
- Dead Air FM adds slow ambient loops (6.4).
- `prefers-reduced-motion: reduce` turns off all movement in both. States show as steady colours instead.

---

## 7. Iconography

- Line icons, 24 px grid, **1.5 px stroke**, round caps and joins, no fill. Lucide (ISC licence) matches; use one set only.
- Default colour `--text-muted`, `--text` on hover, `--accent` only when the icon is the action itself.
- Sizes: 16 px inline and in buttons, 18–20 px in the header.
- The only filled shape in the UI is the lamp (6–8 px circle), and it always means state.
- No emoji anywhere.

---

## 8. Logo

Files are in `assets/brand/` (see `assets/brand/BRAND.md`).

- **Mark:** a ghost with **one** round lamp eye and a voice-meter tail. The eye uses `--mark-eye`: blue in the parent, valve amber in Dead Air FM. It is the same lamp as the dot on the i in the wordmark.
- **Animated:** speaking (body), heard you (eye flashes orange, tail goes flat), listening (eye turns blue), back to speaking. 5.2 s loop.
- **Header:** static lockup (mark + wordmark) followed by a `DOCS` tag. Dead Air FM's header uses the mark with its own serif name instead of the wordmark.
- **Favicon:** `favicon.svg` + `favicon.ico`; Apple touch icon `png/ghostline-icon-180.png`. **Share image:** `png/ghostline-og.png` (1200 × 630).
- Never give the ghost two eyes, a wavy hem or purple: those belong to Kiro's mascot.

---

## 9. Docs site: ghostline.dualmindlab.tech (parent)

### 9.1 Recommended stack (decide in a spec, then keep)

- **Framework:** Astro with Starlight. Restyle it with this token file through Starlight's custom CSS option; check Starlight's current docs for its own CSS variable names before mapping them.
- **Hosting:** Cloudflare Pages (or Workers static assets), CNAME `ghostline` → the Pages project on the `dualmindlab.tech` zone.
- **Code highlighting:** Shiki (via Expressive Code); map scopes to `--syn-*` as in 3.4.

These are recommendations. If you pick another stack, the tokens and layouts below still apply.

### 9.2 Site map

```
/                         Home
/docs/                    Introduction
/docs/quickstart/
/docs/concepts/
/docs/core/turn-policy/   /barge-in/   /history/   /tracing/
/docs/doorways/browser/   /simulator/  /exotel/    /twilio/
/docs/providers/sarvam/   /elevenlabs/ /workers-ai/
/docs/apps/dead-air-fm/   /missed-call-oracle/
/docs/kiro/power/         /steering/   /specs/
/bench/                   Metrics, method, link to the Hugging Face dataset and Space
/brand/                   Logo downloads, colours, usage rules
```

Sidebar groups, in order: Get started · Core · Doorways · Providers · Apps · Kiro · Bench.

### 9.3 Home page (top to bottom)

1. **Header** (9.5).
2. **Hero**, two columns. Left: `label` eyebrow "Open source · Cloudflare Workers · Built with Kiro", `display-xl` headline, `body-l` lead in `--text-muted`, primary + secondary buttons, install code block. Right: a `--surface` panel (`--radius-lg`) with a soft `--glow` behind the animated mark (300 px) and three lamps (Speaking, Heard you, Listening) in sync with the logo.
3. **01 · The difference:** naive pipeline vs Ghostline, two cards. The naive card has an `--alarm` border and highlights unheard words in `--alarm-soft`; the Ghostline card strikes them through in `--text-faint`.
4. **02 · The skeleton:** three cards (Turn policy, Barge-in controller, Latency tracer).
5. **03 · Two apps:** Dead Air FM and Missed-Call Oracle cards.
6. **04 · The numbers:** results table with "A dash means not measured yet. Targets are goals, not results."
7. **05 · Built with Kiro:** four rows (Specs, Steering, Hooks, Power + MCP).
8. **Closing CTA panel** and **footer**.

Section rhythm: `--space-24` between sections on desktop, `--space-16` on mobile. Section titles are `display-l` with a `label` eyebrow "0N · Name".

### 9.4 Doc page

Three columns (5.5). Content: breadcrumb (`caption`, muted) → `h1` → lead (`body-l`, muted) → body → Previous / Next cards. On-this-page lists `h2`s only; the current one gets a 1 px `--accent` left rule.

### 9.5 Components

| Component | Spec |
|---|---|
| Header | `--size-header`, `--surface`, 1 px `--line` bottom border, sticky. Lockup + DOCS tag; nav (active: `--text` with a 2 px `--accent` bottom rule); search (240 px, `--surface-raised`, `⌘K` kbd); GitHub; theme toggle. |
| Sidebar | Group titles in `label` muted. Items `body-s` `--text-muted`, 36 px, `--radius-md`. Active: `--text`, 500, `--surface-raised`, a 6 px `--accent` lamp, `aria-current="page"`. |
| Button, primary | `--accent-fill`, `--on-accent` text, Geist 600 15 px, 44 px tall, `--radius-md`. Hover `--accent-fill-hover`. |
| Button, secondary | Transparent, 1 px `--line-strong` border, `--text`. Hover `--surface-raised`. |
| Input | 44 px, `--surface-raised`, 1 px `--line-control`, `--radius-md`; label above (Geist 500 14 px), helper `caption` below. |
| Callout | `--surface`, 1 px `--line`, `--radius-lg`. A 6 px dot (note muted, tip `--state-listening`, warning `--amber`, danger `--alarm`), bold title, body in `--text-muted`. No coloured fills, no left stripe. |
| Code block | `--code-bg`, 1 px `--code-line`, `--radius-lg`. 40 px top bar: filename or tabs, language, Copy. Body 14/24 Geist Mono, line numbers (or `$` for shell) in `--syn-comment`, never wraps. |
| Table | 1 px `--line` wrapper, `--radius-lg`, `--surface`; headers in `label` over a `--line-strong` rule; numbers right-aligned in `data`. |
| Kbd | Geist Mono 11 px, `--surface-raised`, 1 px `--line-strong`, `--radius-xs`. |
| Lamp + label | 6–8 px circle in a state colour + `label`, 8 px gap. |
| Search dialog | `--surface-overlay`, `--shadow-overlay`, `--radius-lg`, 640 px, opens with `⌘K` / `Ctrl K` / `/`. |

### 9.6 Theme behaviour

- Default: Night. If the system prefers light and the visitor has not chosen, use Day (`tokens.css` handles this).
- The toggle sets `data-theme="dark"` or `"light"` on `<html>` and remembers it in `localStorage`, set by an inline script in `<head>` before first paint.
- `<meta name="theme-color">`: `#0a0a0a` on Night, `#ffffff` on Day.

### 9.7 Meta and sharing

- `og:image`: `https://ghostline.dualmindlab.tech/og.png`; `twitter:card` = `summary_large_image`.
- Titles: `<Page> · Ghostline docs`. Home: `Ghostline: voice agents that know when to shut up`.

---

## 10. Dead Air FM (child)

Set `data-theme="deadair"` on the app root. Everything below is built from tokens; the overrides in 3.2 do the rest.

### 10.1 Screens

| Screen | What is on it |
|---|---|
| Tune in | Mark with `--glow`, a `dial` line "99.3 FM · 03:12 AM · …", `station` heading "Dead *Air* FM" (Air in `--state-speaking`), lead in `--text-muted`, a bone pill "Tune in" + a ghost pill "How it works", a mic note, and the dial with the needle sweeping onto 99.3. |
| On air | Header (mark, serif name, LIVE pill, clock, settings). Left: the host (mark 132 px, lit ON AIR sign, name in serif, segment line), the live `transcript` (spoken words `--text`, current word `--state-speaking` with glow, unspoken words `--text-faint`), the host waveform. Right (360 px): Tonight's schedule, "Cut in anytime", This session (numbers `—`). Bottom dock: dial + mic button. |
| You cut in | ON AIR goes dark, the screen edge glows `--state-heard` for at most 150 ms then holds faintly, the mark's tail goes flat and its eye turns `--state-heard`. Unheard words are `<del>` in `--text-faint` with an orange strike. A "Heard you · stopped in — ms" pill. The caller's words in a `--accent-soft` bubble with a blue waveform. The mic button fills with `--state-listening` and pulses. |
| Mobile | Same as On air in one column: header, mark + ON AIR, transcript (`transcript-s`), waveform, dial, a full-width "Just talk to cut in" pill. |

### 10.2 Child components

| Component | Spec |
|---|---|
| ON AIR sign | `dial` style, 0.32em tracking, 1.5 px border, `--radius-sm`. Lit: `--state-speaking` text and border with a soft glow; flickers once every 7 s. Dark: `--text-faint` text, `--line-strong` border. |
| Dial | 88–108 MHz scale, a major tick each 1 MHz, minor each 0.2, labels every 2 MHz (every 4 on mobile) in Geist Mono 11 px. Station names in serif italic. Needle 2 px `--state-speaking` with glow; drifts 3 px every 6 s. |
| Transcript | `transcript` / `transcript-s`. Three colours only: spoken, current, unspoken. Never animate words already spoken. |
| Mic button | 76 px circle. Idle: 1.5 px `--line-strong` ring. Listening: `--state-listening` fill, `--bg` icon, a pulse ring. |
| Caller bubble | `--accent-soft` fill, 1 px blue border at 35 %, `--radius-lg`, "YOU" in `dial` blue, words in `transcript`. |
| Cards | `--surface` at low opacity over the atmosphere, 1 px `--line`, `--radius-lg`. Titles in `transcript-s`, labels in `dial`. |
| Buttons | Pills (`--radius-pill`). Primary: `--accent-fill` (bone) with `--on-accent`. Secondary: 1 px `--line-strong`. |

### 10.3 Atmosphere layers (bottom to top)

1. `--bg`.
2. Light: a cold moon glow top-right (blue-grey at about 15 %) and a warm valve glow bottom-left (orange at about 10 %).
3. Fog: two soft blobs along the bottom edge, blurred, drifting 3 % over 18 s.
4. Content.
5. Vignette: radial, edges darkened to `--fx-vignette`.
6. Scanlines: 1 px dark line every 3 px at `--fx-scanline`, multiply.
7. Grain: fractal noise at `--fx-grain`, jittering in 6 steps per 0.9 s.

Layers 2–7 are `pointer-events: none` and `aria-hidden`. Text must still pass contrast **with** the layers on.

### 10.4 Motion (child only)

ON AIR flicker (7 s), waveform bars (1.2 s alternate), mark halo breathing (3.2 s), needle drift (6 s), needle sweep on tune-in (5 s), mic pulse (1.6 s), LIVE dot (1.4 s), fog (18 s), grain (0.9 s). All stop under `prefers-reduced-motion`.

### 10.5 Not allowed in the child

Pumpkins, bats, skulls, cobwebs, blood, purple, slime green, cartoon ghosts, creepy fonts (dripping, scratchy), jump scares, sound that plays before the visitor presses Tune in.

---

## 11. Missed-Call Oracle (parent)

The browser simulator uses the parent theme unchanged (Night by default). Its parts: a phone keypad (the pressed key in `--accent-fill`), a call timer in `data`, lamps for the four states, and a transcript where Hindi and Gujarati lines carry `lang="hi"` / `lang="gu"`. Language names are always written in their own script: हिन्दी, ગુજરાતી, English.

---

## 12. Accessibility checklist

- [ ] Text contrast 4.5 : 1 in all three themes (tokens pass; do not override colours).
- [ ] Every interactive element is a real `<a>`, `<button>` or `<input>` with a visible focus ring: 2 px `--focus`, 2 px offset.
- [ ] Icon-only buttons have `aria-label`.
- [ ] Colour is never the only signal: lamps have labels, callouts have titles, struck words have a "Never played" note.
- [ ] Skip link "Skip to content" first; headings in order; one `h1`.
- [ ] Scrolling code blocks are focusable (`tabindex="0"`).
- [ ] `prefers-reduced-motion` respected everywhere, including all Dead Air FM atmosphere.
- [ ] Hindi and Gujarati content carries `lang`.

---

## 13. Voice and microcopy

Short, dry, a little haunted. Plain English. No hype, no exclamation marks, no emoji. The docs are calm; Dead Air FM may be a little more theatrical.

| Situation | Write |
|---|---|
| Empty state | Dead air. Say something. |
| Barge-in | Heard you. |
| Error | Line dropped (speech service). Reconnecting… |
| Not measured | — ms |
| Primary CTA (docs) | Read the quickstart |
| Primary CTA (Dead Air FM) | Tune in |
| 404 | This page gave up the ghost. Back to the docs. |

---

## 14. Avoid

Purple or blue-purple gradients · glassmorphism · neon in the parent · sparkle "AI" icons · emoji · glowing orbs · cards with a coloured left border · uppercase labels in the parent · Halloween props anywhere · fake typing dots · unlabelled numbers · orange used for decoration · a second eye on the ghost · `0` as a placeholder for a number.
