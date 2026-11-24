# Ghostline brand kit

The mark is a ghost with one round lamp eye and a voice-meter tail. The single lamp eye is what keeps it ours: Kiro's own mascot is a ghost with two oval eyes, so never give this one two eyes. The lamp is the same lamp as the dot on the i in the wordmark.

One loop (5.2 s) acts out the product:

| Time | State | What you see | Meaning in the app |
|---|---|---|---|
| 0–45 % | Speaking | Tail bars bounce like a level meter; the eye is bone | Bot is talking |
| 45–56 % | Heard you | The eye flashes orange | Real speech confirmed (barge-in) |
| 50–56 % | Dead stop | The tail flatlines into one even hem, starting from the middle | No more audio for that turn |
| 58–84 % | Listening | The eye turns blue and pulses | Waiting for the user |
| 84–100 % | Back | The tail drops back down | Next turn |

The still logo is the full tail with a blue eye.

## Files

| File | Use it for |
|---|---|
| `ghostline-icon-animated.svg` | Animated app icon (dark rounded tile). Best for the README top and the site |
| `ghostline-icon.svg`, `png/ghostline-icon-*.png` | Static app icon, PWA icons (192, 512), Apple touch icon (180) |
| `ghostline-mark-animated-on-dark.svg` / `-on-light.svg` | Animated mark with a transparent background |
| `ghostline-mark-on-dark.svg` / `-on-light.svg` | Static mark with a transparent background |
| `ghostline-lockup-animated-dark.svg` | Mark + wordmark on a dark panel, animated (the i-dot flashes orange, then turns blue) |
| `ghostline-lockup-animated-on-light.svg` | Same, ink colors, transparent background |
| `ghostline-lockup-dark.svg`, `ghostline-lockup-on-light.svg`, `png/ghostline-lockup-*.png` | Static lockups |
| `ghostline-banner-animated.svg` | 1280×440 README banner with state lamps |
| `ghostline-banner.svg`, `png/ghostline-banner@2x.png` | Static banner |
| `ghostline-banner.mp4` | Banner animation, 3 loops, for social posts and the video intro |
| `ghostline-icon.gif` | 256 px animated icon for places that block SVG (Devpost gallery, chat apps) |
| `ghostline-og.svg`, `png/ghostline-og.png` | 1200×630 share image (Open Graph / Twitter card) |
| `favicon.svg`, `favicon.ico`, `png/favicon-*.png` | Simplified ghost (three tail bars, bigger eye) that still reads at 16 px |

Text in every SVG is converted to outlines, so no font needs to be installed to view them.

## README snippet (switches with GitHub's light and dark theme)

```html
<p align="center">
  <picture>
    <source media="(prefers-color-scheme: dark)" srcset="assets/brand/ghostline-lockup-animated-dark.svg">
    <img src="assets/brand/ghostline-lockup-animated-on-light.svg" alt="Ghostline" width="420">
  </picture>
</p>
```

Or use the banner: `<img src="assets/brand/ghostline-banner-animated.svg" alt="Ghostline: voice agents that know when to shut up" width="100%">`

## Site head

```html
<link rel="icon" href="/favicon.svg" type="image/svg+xml">
<link rel="icon" href="/favicon.ico" sizes="any">
<link rel="apple-touch-icon" href="/png/ghostline-icon-180.png">
<meta name="theme-color" content="#0a0a0a">
<meta property="og:image" content="https://YOUR-DOMAIN/ghostline-og.png">
<meta name="twitter:card" content="summary_large_image">
```

## Colors and type

The full system (Graphite Night and Day, the Dead Air FM child theme, every token, the docs-site layouts) is in `docs/DESIGN.md` and `design/tokens.css`. The brand colors used in these logo files:

| Name | Hex | Job |
|---|---|---|
| Ink (`gray-900`) | `#0a0a0a` | Night ground, the icon tile |
| Panel (`gray-850`) | `#111111` | Raised surfaces |
| Line (`gray-750`) | `#262626` | Rules, dividers |
| Ghost (`gray-150`) | `#ededed` | The ghost, main text on Night |
| Muted (`gray-400`) | `#a1a1a1` | Labels |
| Signal blue (`blue-400`) | `#5b9bff` | The eye, listening, on Night |
| Signal blue on light (`blue-600`) | `#1f5fd6` | The eye, listening, on Day |
| Orange (`orange-400`) | `#ff6a33` | Interruption only |
| Valve (`valve-400`, Dead Air FM only) | `#ffb257` | The Dead Air FM eye and ON AIR glow |

Type: the wordmark is outlined Space Grotesk (artwork, not live text). Everything else: Geist (text and headings), Geist Mono (code and numbers). Dead Air FM adds Instrument Serif. All free on Google Fonts.

## Rules

Do:
- Use the animated mark where it can move (README, site header, video intro) and the still mark (blue eye) elsewhere.
- Keep orange for interruption and blue for listening, in the logo and in the product. Dead Air FM's mark uses a warm valve-amber eye; that is the only variant.
- Switch to the favicon mark under 32 px.
- Keep one bar width of clear space on every side.

Don't:
- Add gradients, glows or outlines.
- Recolor the bars or the lamp for decoration.
- Give the ghost two eyes, a wavy hem, or purple. Those are Kiro's mascot, not ours.
- Speed up the loop.
- Stretch, rotate, or place the mark on busy photos.
