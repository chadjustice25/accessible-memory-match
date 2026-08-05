# Accessible Memory Match

A memory-match card game rendered on HTML Canvas — a single self-contained
HTML file, no dependencies, no build step.

**Status: in progress.** The rendering engine, data model, and responsive
grid layout are built and working. Game rules (matching logic, move
counter, win state), keyboard navigation, and the accessible live region
are designed but not yet implemented — see "What's next" below.

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

## Accessibility approach (planned, not yet implemented)

Canvas content is pixels, not DOM elements — screen readers and other
assistive technology have nothing to read, and there's no natural place
for keyboard focus to land. The plan to address that:

- **Keyboard navigation** — arrow keys to move a focus position across
  the grid and Enter/Space to flip the focused card, since there's no
  underlying DOM element for the browser to focus natively. This needs
  a visible on-canvas focus indicator, since there's no built-in
  `:focus` styling either.
- **A visually-hidden live region** — an off-screen DOM element (not
  drawn on the canvas) that announces card values on flip, match
  results, and the win state, so the game state is available to
  screen reader users as it changes.

## What's next

- Match-checking game logic (currently any card flips freely; capping
  it at two face-up cards and checking for a match is not yet wired up)
- Move counter and win state
- Keyboard navigation and the live region described above
- Restart control
