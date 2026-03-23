# FOR_Jan: HoursToBuy — The Price Tag for Your Life

> *"That MacBook doesn't cost $1,499. It costs you 100 hours of work. Still want it?"*

---

## 1. Project Overview

HoursToBuy is a single-file web tool that converts any price into hours, days, and weeks of your working life. You type your hourly wage and the item's price, and it shows you — visually — how much of your time that purchase really costs. Animated bars fill up, day-blocks stack one by one, and a verdict tells you whether it's a "No brainer" or a "Major commitment."

The whole point is psychological: we're numb to dollar amounts, but watching 12.5 workday blocks appear on screen actually hits different. It's a spending gut-check disguised as a calculator.

Built as a zero-dependency single HTML file — vanilla JS, CSS animations, Google Fonts. No build tools, no npm, no login. Works offline, runs anywhere.

---

## 2. Technical Architecture

This is about as simple as architecture gets — one file, zero network calls, everything inline:

```
┌──────────────────────────────────────────────┐
│              index.html (27KB)               │
│                                              │
│  ┌─────────────────────────────────┐         │
│  │  <style>                        │         │
│  │  - CSS variables (color system) │         │
│  │  - Outfit + JetBrains Mono      │         │
│  │  - Animations (scaleX, scale)   │         │
│  │  - Responsive + reduced motion  │         │
│  └─────────────────────────────────┘         │
│                                              │
│  ┌─────────────────────────────────┐         │
│  │  <body>                         │         │
│  │  - Form (wage + price inputs)   │         │
│  │  - Popular items (from config)  │         │
│  │  - Results (hero + bars +       │         │
│  │    blocks + verdict)            │         │
│  └─────────────────────────────────┘         │
│                                              │
│  ┌─────────────────────────────────┐         │
│  │  <script>                       │         │
│  │  - POPULAR_ITEMS config array   │         │
│  │  - VERDICTS threshold table     │         │
│  │  - SVG icons (no emojis)        │         │
│  │  - Input parsing (lenient)      │         │
│  │  - Render functions (hero,      │         │
│  │    bars, blocks, verdict)       │         │
│  │  - Event delegation             │         │
│  └─────────────────────────────────┘         │
└──────────────────────────────────────────────┘
```

**Key design decisions:**
- **Single HTML file constraint:** No build tools, no external dependencies beyond Google Fonts. You can literally email this file to someone and it works.
- **GPU-accelerated animations:** All bar fills use `transform: scaleX()` instead of animating `width`, which would cause layout reflow and jank on mobile.
- **Config-driven popular items:** The `POPULAR_ITEMS` array at the top of the script makes adding items trivial — one line per item, categories derived from the data.
- **Event delegation:** One click listener on the container instead of N listeners on individual chips. Handles dynamically generated elements automatically.

---

## 3. Codebase Structure

```
HoursToBuy/
└── index.html          # The entire app (27KB, ~900 lines)
    ├── CSS section      # Custom properties, layout, animations
    │   ├── Color system (severity-low → severity-critical)
    │   ├── Typography (Outfit display, JetBrains Mono data)
    │   ├── Input styling with currency symbol overlay
    │   ├── Bar tracks + fill animations (scaleX)
    │   ├── Day block grid + staggered pop-in
    │   ├── Verdict (border-top divider, not a card)
    │   ├── Category groups for popular items
    │   └── prefers-reduced-motion + responsive breakpoints
    ├── HTML section
    │   ├── Header (left-aligned, asymmetric)
    │   ├── <form> wrapper (Enter-to-submit)
    │   ├── Popular items container (rendered by JS)
    │   └── Results: hero number → bars → blocks → verdict
    └── JS section
        ├── Constants: POPULAR_ITEMS, VERDICTS, ICONS (SVGs)
        ├── Helpers: parseInput, formatNumber, formatPrice
        ├── renderPopularItems() → generates category chips
        ├── Validation: wage > 0, price >= 0, lenient parsing
        ├── Render: renderHero, renderBars, renderBlocks, renderVerdict
        └── Events: form submit + chip click delegation
```

**Data flow:** Input → parseInput (strips $, commas, whitespace) → validate → calculate (hours = price / wage) → render all four result sections with staggered animations.

---

## 4. Technology Choices

| Technology | Why | Alternatives Considered |
|---|---|---|
| Vanilla JS | Zero dependencies, works offline, single file | React (rejected — overkill for a calculator) |
| CSS `@keyframes` + transitions | Hardware-accelerated, no JS animation libraries needed | Framer Motion, GSAP (rejected — requires npm) |
| `transform: scaleX()` for bars | GPU-accelerated, 60fps on old phones | Animating `width` (rejected — triggers layout reflow) |
| Google Fonts (Outfit + JetBrains Mono) | Distinctive typography without self-hosting | System fonts (rejected by taste-skill — too generic) |
| `inputmode="decimal"` | Shows numeric keypad on mobile automatically | `type="number"` (rejected — shows spinner arrows, blocks $ input) |
| `<form>` with submit event | Free Enter-key submission, better semantics | Manual keydown listener (rejected — reinvents HTML) |
| Event delegation on container | One listener handles all dynamic chips | querySelectorAll (rejected — doesn't handle dynamic elements) |
| SVG icons for verdicts | Clean, scalable, no emoji rendering issues | Emojis (rejected by taste-skill anti-emoji policy) |

---

## 5. Lessons Learned

**The taste-skill changes everything about how an app feels.**
We built the functional version first with system fonts, centered layout, and emojis. Then applied the taste-skill and the same calculator suddenly felt premium — Outfit + JetBrains Mono, left-aligned asymmetric layout, SVG icons, desaturated warm palette, subtle grain texture. The code barely changed; the perception completely transformed.
Rule: Design polish is not cosmetic — it's the difference between "meh" and "whoa."

**Lenient input parsing prevents user frustration.**
Users copy-paste prices from shopping sites. Those prices include `$`, commas, and spaces. A simple regex strip (`/[$,\s]/g`) handles all of it. Three lines of code, massive UX improvement.
Rule: Always strip formatting characters from numeric inputs. The user shouldn't have to think about formatting.

**`prefers-reduced-motion` is trivial to add and matters a lot.**
Five lines of CSS disable all animations for users with vestibular disorders. The bars and blocks still show — they just appear instantly. Caught this during the engineering review, not the design phase.
Rule: Add `@media (prefers-reduced-motion: reduce)` to any animation-heavy page. It costs nothing.

**Config-driven UI is worth the small upfront cost.**
The original had 4 hardcoded chips. Switching to a `POPULAR_ITEMS` config array + dynamic rendering was ~40 lines of JS, but now adding an item is literally one line. The config also serves as documentation — you can see every item at a glance.
Rule: If you have more than 3 hardcoded items, make it config-driven.

**Day-block cap prevents DOM explosion.**
At $0.01/hr wage and $100K price, you'd get 1.25 million days = 1.25 million DOM elements. The 30-block cap with a "+N more days" overflow label prevents this entirely.
Rule: Always cap dynamically generated DOM elements. Think about what happens with extreme inputs.

---

## 6. War Stories

**The office hours session that shaped everything.**
This project started with a `/office-hours` session in builder mode. The key moment was when the question "What's the coolest version of this?" came up, and the answer was "visual wow factor" — not features, not sharing, not social. That single answer shaped the entire architecture: animated bars, stacking day-blocks, the emotional verdict. Seven competitors were researched beforehand (Work Days to Buy, TimePrice, Time Value, SpendYourHours, Work2Buy, Time Pricer, Worth My Time) — every single one just shows a number. The visual approach is the only differentiator.

**The engineering review that caught 6 issues before a single line was written.**
The `/plan-eng-review` session added: `inputmode="decimal"` for mobile keypads, `prefers-reduced-motion` for accessibility, lenient input parsing, `<form>` wrapping for Enter-to-submit, explicit threshold boundaries (no off-by-one at exactly 4 hours), and GPU-accelerated `transform: scaleX()` instead of animating `width`. All of these were "obvious in hindsight" — the kind of thing you only catch when someone forces you to think about edge cases before coding.

**The browse tool couldn't open `file://` URLs.**
When trying to screenshot the wireframe, the gstack browse binary blocked `file://` protocol. Had to spin up a `python3 -m http.server` to serve it via localhost. Small friction, but a reminder that headless browsers have security restrictions even for local files.

**The taste-skill anti-emoji rule.**
The original design used emojis for verdicts (sunglasses face for "No brainer," grimacing face for "Think twice"). The taste-skill has a strict anti-emoji policy, so these got replaced with clean inline SVGs — checkmark, thumbs-up, pause, alert triangle, flame. Honestly, the SVG icons look more professional. The emojis were fun but felt like a toy; the icons feel like a tool.

---

## Quick Reference

**To add a new popular item**, edit the `POPULAR_ITEMS` array near the top of `index.html`:
```js
{ name: 'Tesla Model 3', price: 38990, category: 'Transport' },
```

**To add a new category**, just use a new category string — it appears automatically:
```js
{ name: 'Rent (monthly)', price: 2000, category: 'Housing' },
```

**Verdict thresholds** (in the `VERDICTS` array):
- < 4 hours: No brainer
- 4-16 hours: Fair trade
- 16-40 hours: Sleep on it
- 40-80 hours: Think twice
- 80+ hours: Major commitment

**Bar fill scaling:** All bars use 80 hours = 100% fill. Items over 80 hours show a full bar.

**Day block cap:** Max 30 blocks displayed. Overflow shows "+N more days."

**To test locally:**
```bash
cd ~/Projects/HoursToBuy
python3 -m http.server 8765
# Open http://localhost:8765
```
