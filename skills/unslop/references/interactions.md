# Fluid Interactions & Tactile Physics

How to give clean, unslopped interfaces the tactile responsiveness, fluid motion, and material depth of Apple-grade software. Distilled from Emil Kowalski's work ([@emilkowalski_](https://twitter.com/emilkowalski_)) and Apple's WWDC design engineering talks (*Designing Fluid Interfaces*, *The Details of UI Typography*).

When `unslop` removes the visual clutter, this guide provides the physical rules for what remains: an interface that feels alive, responds instantly, and behaves like real materials.

---

## 1. Response: Kill Input Latency

Lag destroys direct manipulation. The interface must feel immediately connected to the user's hand or pointer.

- **Feedback on pointer-down, not click:** Highlight or depress a button or card the instant it is pressed (`:active` state). Waiting for touch-up or click to provide feedback makes an app feel sluggish.
- **Physical press state:** Subtle scale and surface step on active:
  ```css
  /* Instant tactile response */
  .button:active, .card-interactive:active {
    transform: scale(0.97);
    transition: transform 100ms ease-out;
  }
  ```
- **Audit hidden delays:** Remove artificial timers, transition delays on input, and unnecessary debounces from interactive controls.
- **Continuous feedback:** For sliders, drag handles, or sheets, update the UI 1:1 with pointer movement throughout the interaction — never animate only when released.

---

## 2. Spring Physics over Fixed Durations

Fixed-duration cubic-bézier animations (`transition: all 300ms ease`) cannot respond naturally to new input or interruptible user actions. Springs can: any new action simply updates the target, and motion stays continuous.

Apple parameterizes springs with two designer-friendly values:
- **Damping ratio:** Controls overshoot.
  - `1.0` = **Critically damped** (zero bounce, smooth settle). **This is the mandatory default for almost all UI.**
  - `0.8` = **Under-damped** (slight bounce). Use *only* when an action carried physical momentum (a flick, a drag release, or a fast gesture). Never add bounce to a menu or popover that simply opened.
- **Response:** How quickly the spring reaches its target, in seconds. Lower = snappier.
  - Default UI elements / reposition: `response: 0.35–0.4s` with `damping: 1.0`
  - Drawers / sheets: `response: 0.3s` with `damping: 0.8`
  - Menus / tooltips: `response: 0.2–0.25s` with `damping: 1.0`

### Web Mapping (Motion / Framer Motion / CSS)
```js
import { animate } from 'motion';

// Default critically damped UI motion (no overshoot, settles gracefully)
animate(element, { y: 0, opacity: 1 }, { type: 'spring', bounce: 0, duration: 0.35 });

// Momentum interaction (card flick or drawer release)
animate(element, { y: target }, { type: 'spring', bounce: 0.2, duration: 0.35 });
```

---

## 3. Interruptibility & Velocity Handoff

> "The thought and the gesture happen in parallel."

- **Never lock input during a transition:** A user must be able to click, reverse, or dismiss a modal or menu while it is still animating.
- **Animate from presentation (current) value:** When interrupting an animation, read the element's current live on-screen transform and animate from there. Starting from the logical target causes a harsh visual jump.
- **Smooth reversal:** If an interaction is cancelled or reversed mid-flight, blend the existing velocity rather than hard-stopping.

---

## 4. Spatial Consistency & Anchored Origins

> "If something disappears one way, we expect it to emerge from where it came."

- **Symmetric entry and exit paths:** A drawer or sheet that slides in from the right must dismiss back to the right. Sliding in from the side and vanishing out the bottom breaks spatial orientation.
- **Anchor origins to triggers:** Menus, popovers, and dialogs should scale from the trigger element that spawned them:
  ```css
  /* Origin-aware popover */
  .popover {
    transform-origin: var(--trigger-x, center) var(--trigger-y, top);
  }
  ```
- **Directional telegraphing:** An opening panel should begin moving in the direction of the user's intent, signaling the final resting state before it arrives.

---

## 5. Apple Materials & Translucency Hierarchy

Translucent materials provide structure without obscuring context. They should feel like physical glass or polished metal catching real light.

- **Floating chrome with backdrop blur:** Navigation bars, sticky headers, and toolbars should float with content visible scrolling underneath:
  ```css
  .floating-bar {
    background: rgba(18, 20, 26, 0.72);
    backdrop-filter: blur(20px) saturate(180%);
    -webkit-backdrop-filter: blur(20px) saturate(180%);
    border-bottom: 1px solid rgba(255, 255, 255, 0.08);
  }
  ```
- **Light-catching top edge (The "Hairline Highlight"):** Dark surfaces gain tactile weight when light catches their top edge:
  ```css
  .card-refined {
    background: #111827;
    border: 1px solid rgba(255, 255, 255, 0.08);
    box-shadow: inset 0 1px 0 rgba(255, 255, 255, 0.12), 0 4px 16px rgba(0, 0, 0, 0.2);
  }
  ```
- **Material weight encodes hierarchy:**
  - Darker, heavier materials frame structural regions (sidebars, backdrops).
  - Lighter, higher-contrast surfaces bring focus to interactive components (cards, dialogs).
  - **Rule:** Never stack light translucent surfaces on top of other light translucent surfaces — legibility collapses.
- **Scroll edge blur instead of hard dividers:** Fade a small gradient/blur mask where content meets floating chrome rather than placing a stark 1px solid line.

---

## 6. Optical Typography: Dynamic Tracking & Inverse Leading

Type behaves differently at different scales. A fixed tracking value is always wrong at some point.

- **Size-specific tracking (letter-spacing):**
  - **Display / KPI figures (>36px):** Negative tracking (`-0.02em` to `-0.03em`). Big glyphs have naturally excessive optical space between them; tightening them makes numbers and headlines punchy and cohesive.
  - **Body text (14–16px):** Neutral tracking (`0em`).
  - **Small meta / labels / badges (<13px):** Slightly positive tracking (`+0.01em` to `+0.025em`) for crystal clarity on small screens.
- **Inverse leading (line-height):**
  - Large display headings need tight leading (`1.05` to `1.15`).
  - Body paragraphs need comfortable leading (`1.45` to `1.6`).
- **Tabular figures for financial & data tables:** Always enforce `font-variant-numeric: tabular-nums` for columns of numbers, dates, and currency, so digits never jitter as values update.

---

## 7. Accessibility & Reduced Motion

Quality software respects platform preferences and motor/vestibular comfort.

- **`prefers-reduced-motion: reduce`:** Replace sliding panels, scaling modals, and spring physics with clean opacity cross-fades (`opacity 150ms ease-out`). Never eliminate the transition entirely — a gentle fade confirms the state change without vestibular stress.
- **`prefers-reduced-transparency: reduce`:** Replace frosted glass (`backdrop-filter`) with solid opaque ramp colors (`#111827` or `#ffffff`).
- **`prefers-contrast: more`:** Increase hairline opacity from 8% to 20%, and raise text contrast.

```css
@media (prefers-reduced-motion: reduce) {
  *, *::before, *::after {
    animation-duration: 0.01ms !important;
    animation-iteration-count: 1 !important;
    transition-duration: 150ms !important;
  }
  .sheet, .modal, .popover {
    transform: none !important;
  }
}

@media (prefers-reduced-transparency: reduce) {
  .floating-bar, .modal {
    backdrop-filter: none !important;
    background: #111827 !important;
  }
}
```

---

## Quick Reference: Fluid Parameters

| Need | Physics / Method | Concrete Value |
| :--- | :--- | :--- |
| **Default UI motion** | Critically damped spring | `damping: 1.0`, `response: 0.35s` (no overshoot) |
| **Momentum / throw** | Under-damped spring | `damping: 0.8`, `response: 0.35s` (slight settle bounce) |
| **Button press** | Instant `:active` feedback | `transform: scale(0.97)`, `transition: transform 80ms ease-out` |
| **Light-catching edge** | Inset top highlight | `box-shadow: inset 0 1px 0 rgba(255,255,255,0.12)` |
| **Display tracking** | Large titles & KPI figures | `letter-spacing: -0.02em`, `line-height: 1.1` |
| **Small label tracking** | Badges & captions | `letter-spacing: +0.015em`, `text-transform: uppercase` (optional) |
| **Floating chrome** | Translucent backdrop | `background: rgba(17,24,39,0.75)`, `backdrop-filter: blur(20px)` |
| **Reduced motion** | Non-vestibular fade | `transition: opacity 150ms ease-out` |

---

Distilled for unslop from Emil Kowalski's design engineering guides and Apple WWDC design principles.
