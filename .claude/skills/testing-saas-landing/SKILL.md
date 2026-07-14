---
name: testing-saas-landing
description: Test the projects/saas-landing static landing page (and similar Tailwind-CDN demo pages) end-to-end after visual/UI changes. Use when verifying look-and-feel redesigns, scroll reveals, hover states, anchor nav, pricing cards, focus rings, reduced motion, or mobile layout.
---

# Testing static landing pages (saas-landing / Tailwind-CDN demos)

These `projects/*/index.html` pages are static HTML with Tailwind via CDN — no build, no backend, no auth. So testing is purely visual/runtime in a browser.

## Serve it
```bash
cd projects/saas-landing        # or the target project dir
python3 -m http.server 8823
# open http://localhost:8823/index.html
```
The dev process is ephemeral — after a VM restart the server is gone; just re-run it. Files on disk persist.

## Browser setup for recording
- Maximize first: `wmctrl -r :ACTIVE: -b add,maximized_vert,maximized_horz` (install wmctrl if missing).
- Navigate via the address bar (computer tool), not devtools.

## What to verify (map to the redesign)
1. Hero theme: gradient headline/stats, glass navbar, accent CTA color, background mesh.
2. Scroll reveals: scroll down; sections should fade in and stay legible (nothing stuck at opacity 0).
3. Hover: card lift + color feedback — subtle in stills, so rely on the recording.
4. Anchor nav: click nav links, confirm URL hash (`#features`, `#pricing`, ...) and the target heading is in view.
5. Pricing: highlighted card gradient/badge, check-icon color; other cards unchanged.
6. Focus rings: press Tab, zoom into the focused control to confirm the `:focus-visible` outline color.
7. Mobile: resize window narrow to test the `md:` breakpoint:
   `wmctrl -r :ACTIVE: -b remove,maximized_vert,maximized_horz; wmctrl -r :ACTIVE: -e '0,40,0,430,900'`
   Expect desktop nav links hidden, content stacked, no horizontal overflow. Re-maximize after.

## Reduced motion (no UI toggle — use CDP)
There is no on-page control for `prefers-reduced-motion`, so verify via the Chrome CDP endpoint (`http://localhost:29229`) with Python `websockets`/`requests`:
- `Emulation.setEmulatedMedia` with feature `prefers-reduced-motion=reduce`
- navigate, then `Runtime.evaluate` computed `opacity` of all `.reveal` — expect all `"1"` (content visible immediately).
- `Page.captureScreenshot` with `captureBeyondViewport:true` for a full-page still.
Do this OUTSIDE the recording (it's shell/CDP, not visible UI).

## Known caveat to always report
Scroll-reveal elements start at `opacity:0` and are un-hidden by JS. With JS disabled the content stays hidden. If not hardened (e.g. `.reveal` scoped under a `.js` class), call this out in the report.

## Gotchas
- `computer` tool needs an `actions` array; `zoom` uses `region` (not `coordinate`).
- Full-page screenshots may show blank reveal sections if elements never entered the viewport — either scroll through them first, or use reduced-motion emulation to force them visible.

## Devin Secrets Needed
None — the page is fully static with no login or backend.
