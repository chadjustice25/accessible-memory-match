# Accessible Memory Match

A memory-match card game rendered on HTML Canvas — a single self-contained
HTML file, no dependencies, no build step.

**Status: feature-complete.** 12-card grid, match-checking, move counter,
win state, keyboard navigation, an accessible live region, and a restart
control are all built and browser-tested.

## Run it

Open `memory-match.html` directly in a browser. That's it — no server,
no install, no build step.

`canvas-demo.html` is an earlier prototype kept alongside it as a
reference; the game itself lives in `memory-match.html`.

## Architecture

- **`devicePixelRatio`-aware sizing** — Canvas has two independent sizes:
  its CSS size (how big it looks) and its buffer size (how many pixels it
  actually has to draw into). Sizing only the CSS dimension leaves the
  browser stretching a lower-res buffer across a high-res screen, which
  is what makes unscaled canvases look blurry on Retina displays. The
  buffer is allocated at `CSS size × devicePixelRatio` and the drawing
  context is scaled to match, so all drawing code can keep working in
  plain CSS units.
- **Delta-time animation loop** — the card flip animates by advancing a
  `progress` value using the elapsed time between frames (`dt`), not a
  fixed increment per frame. That keeps the animation speed consistent
  regardless of display refresh rate, and `dt` is clamped so a
  backgrounded tab can't cause the animation to jump.
- **Responsive grid layout** — the 12 cards (6 pairs, shuffled) sit in a
  fixed 4×3 grid whose card size is recomputed from the available
  viewport width on load and on every resize, so the layout scales down
  cleanly on narrow/mobile screens without changing the grid shape.
- **Manual hit-testing** — Canvas has no DOM elements and no
  `event.target` to inspect. Clicks are converted from viewport
  coordinates to canvas coordinates and checked against each card's
  computed bounds to determine what was clicked.
- **The flip effect** — a horizontal scale from full width to zero and
  back (swapping which face is drawn at the midpoint), not a true 3D
  rotation. It reads as a flip at this speed and is far cheaper than a
  real 3D transform.

## Accessibility approach

Canvas content is pixels, not DOM elements — screen readers and other
assistive technology have nothing to read, and there's no natural place
for keyboard focus to land. This is addressed with:

- **Keyboard navigation** — arrow keys move a focus position (`focusIndex`,
  an index into the card array) across the grid, clamped at the edges; Enter
  or Space flips the focused card. A focus ring is drawn on canvas by hand,
  shown only when `canvas.matches(':focus-visible')` — the browser's own
  "was this focus from a keyboard" heuristic — so a mouse/touch tap doesn't
  also show it.
- **A visually-hidden live region** — an off-screen (not `display:none`)
  `aria-live="polite"` DOM element that announces a flipped card's value in
  plain English ("King of Hearts"), the match/no-match result, and the win
  state as they happen.
- **A restart control** as a real `<button>`, getting native focus and
  keyboard support for free — unlike the canvas, which needed all of the
  above built by hand.
