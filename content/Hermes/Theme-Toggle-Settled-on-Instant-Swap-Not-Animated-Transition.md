## Symptom

On a static tool site (ToolBox, Apple‑style design), toggling light‑dark mode worked smoothly on desktop but felt "super laggy" on phones. The background and tool cards switched almost immediately, while the text color on elements such as card titles (`JSON Formatter`) and the small SVGs in the top bar visibly lagged behind. After shortening the transition from 0.3 s → 0.18 s → 0.05 s the mismatch shrank but never disappeared; only when the transition was set to `0s` (an instant in‑frame swap) did background and text land in the same frame.

## Principle

### Why the first attempt used a global animated transition

The original goal was **"no flicker"**, and the established playbook from earlier notes applied the VitePress‑style blanket rule:

```css
*, *::before, *::after {
    transition: background-color .3s ease, color .3s ease, border-color .3s ease;
}
```

On desktop this is perfect: every element animates with the same duration and timing function, so nothing snaps out of sync. The rule was never challenged because the desktop demo looked great.

### Why animation can never be *perfectly* in sync on mobile

Even when every element declares the identical `transition-duration`, two different classes of work run in the same frame:

- `background-color` can be interpolated by the **compositor / GPU**.
- `color` (text glyphs) must be **re‑rasterized on the main thread**, which is CPU‑bound.

On a busy low‑end phone, the text repaint frames are scheduled after the background frames, so text visually trails the background by one or two frames no matter how short the duration is. 0.05 s (~3 frames) minimises the gap but does not remove it.

On top of that, the blanket `*` rule makes **thousands of nodes animate at once**, and the sticky header's `backdrop-filter: blur(20px)` re‑blurs the layer every frame during the swap — a major GPU cost on phones that made the "lag" dramatic rather than subtle.

### The constraint changed, so the solution changed

- Original constraint: **eliminate flicker** → requires an animated transition.
- Final constraint: **eliminate perceived delay + absolute text/background synchronization, on weak mobile GPUs** → the transition itself became the obstacle.

Conclusion: a `0s` "insta‑snapshot" swaps every color inside one rendering frame. Text and background are guaranteed to land simultaneously because there is nothing to schedule, and the costly per‑frame work disappears. This is the most extreme form of the "unified transition" family and the only one that is *architecturally* synchronous.

## Solutions

### Option 1 (initial): global unified transition, 0.3 s → 0.18 s → 0.05 s

Follows the established flicker‑free pattern. Desktop fine; mobile keeps a trailing‑text sensation. Used `--theme-dur` as a single knob (var) so only one value needs editing. Accepted trade‑off: duration shortening is a band‑aid, not a fix.

### Option 2 (intermediate): strip the expensive animatable properties

```css
*, *::before, *::after {
    transition: background-color var(--theme-dur) var(--ease),
                color var(--theme-dur) var(--ease),
                border-color var(--theme-dur) var(--ease);
}
```

- Removed `box-shadow` from the global transition (expensive to animate per‑frame; hover shadow lives only in a `(min-width: 641px) and (hover: hover) and (pointer: fine)` rule so touch devices skip it).
- Disabled `backdrop-filter` on phones/narrow screens:

```css
@media (max-width: 640px) {
    :root { --theme-dur: 0.08s; }
    .site-header { backdrop-filter: none; -webkit-backdrop-filter: none; }
}
```

This made the swap much faster but text still trailed on real hardware.

**Watch‑out:** `(pointer: coarse)` emulation in headless Chromium does not reliably match real touch input; a `max-width: 640px` fallback was added precisely because it is deterministic. Avoid relying on pointer emulation for mobile‑only behavior decisions.

### Option 3 (final): instant swap, `--theme-dur: 0s`

```css
:root { --theme-dur: 0s; }
```

Every color including text changes in the same frame — no scheduling, no laggy middle frames. Interaction animations that must stay smooth are kept separate on the same element by listing `transform`/`opacity` with their own duration while color uses `var(--theme-dur)` (which is now `0s`), e.g. the card hover lift still animates 0.2 s while its colors snap instantly.

Caveat: you lose the soft fade / interpolation that users may expect on desktop retina screens; accept the trade‑off when "in‑frame sync" and mobile performance outrank polish.

## Summary

- Original fix targeted **flicker** (needs animation); final requirement is **in‑frame sync on weak phones** (animation is the enemy). Revisit the goal before re‑tuning the numbers.
- Text `color` animation is QoS‑limited versus `background-color`, so identical durations still desync on low‑end CPUs. The only bulletproof sync is a `0s` swap.
- Mobile‑first rules for full‑page color swaps: keep it instant, drop `backdrop-filter`, drop `box-shadow` from animations; keep animations only for cheap properties (`transform`, `opacity`) used as interaction feedback.
- When the blanket `*` rule fights a specific element, override it with a proper transition list (the property is replaced wholesale, not merged) — never `transition: none !important`.
- Related notes: `Causes-of-Page-Flicker-in-LightDark-Mode-Toggle-and-UnifiedTransition-Solution.md`, `Preventing-Night-Mode-Flash-on-Refresh-in-Firefox.md`.