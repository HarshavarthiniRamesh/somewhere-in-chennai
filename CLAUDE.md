# Somewhere in Chennai

A single-page Astro site: a curated, filterable table of Chennai communities and
meetups. Designed by Harsha, built with Claude Code. "Somewhere in Chennai" is
the project's name; the site's own title is **"Not home. Not work. / But the
secret third thing."** (two lines, hard break, Figma `956:142`) since
2026-10-10, after a day as "Chennai Communities Guide". The tagline is "A
curated list of communities and activities in Chennai to help you find your
people. I hope you find what you're looking for here :)".

## Commands

```sh
npm install      # first time, or after pulling on a new machine
npm run dev      # local dev server at localhost:4321
npm run build    # production build into dist/
npm run preview  # preview the production build
```

Requires Node >= 22.12.0.

## Shape of the project

Deliberately minimal — three files hold everything. There is no components
directory and no UI framework. Keep it that way unless Harsha asks otherwise.

| File | Holds |
| :--- | :--- |
| `src/pages/index.astro` | All markup, plus every interaction in one inline `<script>` |
| `src/data/communities.js` | The community entries and the `filterCategories` pill order |
| `src/styles/global.css` | Design tokens and all styling |

Plus `public/about/`: the three polaroid photos the About letter shows,
exported from Figma `973:24` and centre-cropped to 240px squares (JPEG). The
signature isn't a file; it's SVG path data in the page's frontmatter.

### `src/data/communities.js`

Each entry is an object with `name`, `description`, `categories` (an array),
`entry`, `frequency`, and `link`. A row with an empty `link` renders as inert —
not clickable, no focus ring — so an entry can be added before its link is known.

`filterCategories` is exported separately and its order matches the Figma
design. It is not derived from the entries, so a new category has to be added
in both places to appear as a pill.

That order is also what packs the pills tidily when they wrap — Figma moved
`Film` up between `Sports` and `History` to fill a row on a 440px screen.
Reordering the list changes how the rows break, so treat it as a layout
decision, not just an alphabetical one.

### `src/pages/index.astro`

The inline script is organised in commented sections: filter pills, column sort,
row click, About letter, submit modal, submit form. `applyFilter()` is shared —
it both hides non-matching rows and renumbers the visible ones, so it must be
called after anything that reorders or re-shows rows.

Row numbers are display-only and always recomputed; don't treat them as IDs.

Filtering is a crossfade, not a cut: a pill click fades the rows on screen
out over 220ms, then `applyFilter(true)` swaps the set and the new rows ease
up 6px into place (WAAPI, 520ms, 30ms stagger capped at ten rows), while
`glideHeight()` animates `<main>` from its old height to its new one over
480ms (overflow-y clipped for the duration). The height glide is the part
that matters most: without it the table jumped size in one frame and
everything below snapped, which is what kept reading as abrupt. A token
drops a swap that a faster second click overtakes.

Filtering never moves the scroll. A shorter list makes the page shorter, and
if it's shorter than where the reader is scrolled to, the browser clamps the
scroll and the page jumps on its own. So the swap pins `<main>`'s height,
then `holdScroll()` keeps a `min-height` just big enough for the current
scroll plus the viewport — the footer lands at the bottom of the screen —
and `relaxHold()` shrinks it as the reader scrolls back up, until the
page's natural height takes over. The natural height is computed from the
table's own height, not by clearing the hold and measuring: clearing it
lets the browser clamp before anything can stop it. Sorting calls
`applyFilter()` without the flag and stays instant. Off under reduced
motion. Earlier cuts (no fade-out, 260ms; then a 160ms fade-out, 420ms, no
height glide) were both too abrupt for Harsha.

Both buttons' labels are windows onto the hero wash (`.btn-ink`, aligned to
page coordinates by the "Submit button ink" script, which handles every
`.btn-ink`): Submit's always, About's only while it fills on hover
(`.is-hover-only`).

The desktop table is one CSS grid on `.community-table`, with `.table-head` and
each `.community-row` adopting its tracks via `grid-template-columns: subgrid`.
Tracks are the Figma widths as ratios — `112/300/468/192/188/180` of 1440 — so
the proportions hold at any width, and `min-width: 1440px` pins them to the
design's real pixels. Below that the table scrolls rather than squeezing, which
is what the tablet frame does too.

Two details that look optional and aren't: `minmax(0, …)` around each `fr`,
because an `fr` track's default floor is `min-content` and with nowrap text
that's the entire string, which would blow the ratios apart on the longest
description; and cells setting no widths of their own, since the tracks own
that. An earlier version used `max-content` tracks with `1fr` on the
description — that column then swallowed all the slack on a wide screen and
left a large gap before Category.

Rows stay real boxes rather than `display: contents`, so they can carry their
own background and border.

Header and body cells share the 20px left padding, but the sort arrow's ink
starts 2.5px into its 11px icon box while text stems start 1.5px in, so the
icon carries `margin-left: -1px` to put the arrow on the column's text line
(measured from pixels in every column). Re-measure if the icon or the body
type changes.

Row hover is a plain `--row-hover` background (2.7% grey) with a 100ms fade,
plus the row's cells easing 4.5px to the right. That's the midpoint Harsha
settled on: the original 3.2% / 6px was a touch strong, and halving it to
2.2% / 3px read as if hover wasn't working. The nudge sits behind `@media (hover: hover) and
(prefers-reduced-motion: no-preference)`, so a tap doesn't strand it and it
disappears for anyone who asked for less motion. A directional wipe with a
multi-row trail was built here and then removed — don't reintroduce it without
asking.

### `src/styles/global.css`

Colors, fonts and layout widths are CSS custom properties on `:root` — use the
tokens (`var(--text-muted)`, `var(--col-name)`) rather than hardcoding values.

Three states, from three Figma frames: desktop 1440 (`846:2`), tablet 744
(`906:2362`) and phone 440 (`906:2926`). Breakpoints are 1120px and 600px, but
the hero type is fluid rather than stepped, so nothing jumps across one.

**Neither line of the title may wrap** — Harsha's hard rule, carried over
from the one-line title. The title is two lines by design, with a `<br>` in
the markup and `white-space: nowrap`, so "two lines" is the ceiling, not a
failure. The size is `min(64px, ceiling)` at every breakpoint: 64px is the
design size (Figma `956:142`, 1.1 line height, 18px to the tagline), and the
ceiling is the widest the longer line can be and still fit on one line with
5% to spare, so no width can wrap it. Ceilings are solved from the line's
width-to-font-size ratio, **11.22 for "But the secret third thing."**
(measured in the browser with the real font, opsz auto):
`0.95 × (vw − gutters − anything else on the row) / 11.22`. That gives
`8.47vw - 4.07px` on the phone (24px gutters), `8.47vw - 6.78px` on the
tablet (40px), and `8.47vw - 19.31px` on the desktop (56px gutters plus the
92px nav and a 24px gap beside it). The cap is the smaller term from 984px
on the desktop row and 836px on the tablet; below that the ceiling rules —
56px at 744, 33px at 440, 23px at 320. Re-measure the ratio and re-solve all
three if the title text, the font or `--page-pad` ever change.

The title sets only `SOFT` and `WONK` and lets `opsz` follow the size, as the
Figma text does; the modal title still pins `opsz 100`. It carries no
optical margin: measured from pixels, the italic N and B start within half a
pixel of the tagline's A. (The "Chennai…" title needed `-0.057em` for its C.)

Tagline is `clamp(16px, calc(0.5747vw + 11.72px), 20px)` at every size — 20px at
1440, 16px from 744 down — except below 600px, where it's set to 15px.
On a phone the pills start 160px below the buttons (Figma `986:19`).

Tablet stacks the header (title, tagline, then both buttons) but keeps the
desktop table and lets it scroll sideways. Only the phone restacks it. 1120px is
where the title, the nav and the page padding stopped fitting on one row
with the original string (the longer title shrinks there instead; see above).

The phone table is a different layout, not the desktop table shrunk: rows
become a grid of `52px 1fr` with the number spanning three stacked lines —
name, description, then a `.cell-meta` line joining category, entry and
frequency as `A, B / Free / Monthly`. `.cell-meta` is built in the markup and
hidden above 600px; `.cell-category`, `.cell-entry` and `.cell-freq` are hidden
below it, and the table drops back to `display: block` so nothing scrolls
sideways on a phone.

The hero wash's mask and veil are percentages, not pixels, so `--sky-reach`
rescales the whole falloff; the script reads the band's rendered height.
Below 600px `--sky-reach` is 640px rather than 948: the phone hero is
about half as tall, and at full reach the tagline sat in the green and was
hard to read. Shortening the band puts it in the cyan-to-white part of the
ramp, as on desktop. That alone stopped the wash at the buttons, so the phone also
has its own stops (`RAMP_PHONE` in the script, mirrored in the phone
`--sky-tint`): the same colours, with the yellow-to-blue blend finishing
above the tagline and the blue fading out over ~240px toward the pills.

Three typefaces, all self-hosted via `@fontsource-variable`: Geist (sans, body),
Geist Mono (labels, numerals), Fraunces (display italic, accents).

`.sky-gradient` is the hero band: a single vertical ramp, `--sky-reach` (948px,
the Figma frame height) tall. The stops come from node `919:11`'s **fill**, not
from sampling a render — Figma writes them as `0deg` on a vertically flipped
container, which nets out to the positions measured from the top.

Order matters and is easy to get backwards: Figma does gradient at full
strength, then dither, then the frame's 80% opacity. The shader follows that.
An earlier version used stops sampled off a rendered frame, which had the
opacity already baked in — so it dithered a faded copy and looked dull, with
the blue channel sitting at 130 instead of 58 at the top of the band.

`--sky-tint` in CSS holds the same ramp pre-composited at 80%, since it's only
the no-WebGL fallback and the Submit button's ink, neither of which dithers.

Read the fill and the effect out of Figma rather than eyeballing a render —
sampling cost several wrong rounds here.

`.sky-push` is a WebGL canvas over the band. It has two jobs, and they're
independently switchable:

- **Dither** (always on), ported from Figma's own effect source rather than
  matched by eye. `get_design_context` on a node carrying a custom effect
  returns that effect's implementation — use it instead of guessing from a
  render. Node `919:11` gave: Blue Noise, pixelSize 2, levels 5, brightness
  100, contrast 1.14, mono false.

  Two things that are easy to get wrong, and were: the 64×64 blue-noise tile
  is *generated*, not shipped — seeded white noise, three passes of
  subtracting a 5×5 box blur, then rank-ordered — and it's sampled **once per
  cell and shared by all three channels**. The colour in the speckle comes
  from the channels sitting at different distances from their quantisation
  step, not from per-channel noise. Cell size is CSS px scaled by DPR, or the
  pattern disappears on a 2× screen.

  One deliberate departure from Figma: `STRENGTH` (0.7) blends every cell
  part-way back toward the smooth ramp after quantising. At full strength
  the 2px speckle sits at the same scale as the 20px tagline and fights it.
  Lowering the whole band's contrast keeps the texture uniform and visible;
  0.45 went too faint in the yellow. A quiet zone that smoothed only the
  patch behind the tagline was tried and rejected — Harsha found the local
  hole looked worse than the noise. Don't bring it back.
- **Mosaic** (`MOSAIC`, currently `false`), after the hero of craft.wild.as,
  which Harsha asked for on 2026-10-08 and parked on 2026-10-10 ("isn't
  working well for me") in favour of the Figma dither. The code is intact
  behind the flag; everything below describes it switched on. Replaces the dither when on: the band becomes
  8px flat tiles (`MOSAIC_CELL`) on a faint grid (`MOSAIC_GAP`,
  `MOSAIC_GRID`). Each tile samples the ramp at its own height, nudged by a
  three-octave value-noise field rotated 30° and squeezed along that axis so
  it reads as diagonal bands (`MOSAIC_AMP`), plus a fixed per-tile scatter
  (`MOSAIC_JITTER`) so single tiles flip early or late as a boundary passes
  — without it whole rows stepped at once and it read as strata. The tile's
  ramp *position* is then snapped to a multiple of `MOSAIC_STEP` (0.03 of
  the band) and the ramp sampled there — about a dozen flat steps from
  yellow through the greens to cyan, pale and white. An earlier cut rounded
  each RGB channel to five levels instead, which collapsed the gradient to
  yellow, one green and cyan; Harsha said it read as two colours. The tiles
  have their own `MOSAIC_STRENGTH` (1, fully flat) and
  `MOSAIC_OPACITY` (1, the ramp at full strength rather than the frame's
  80%): a first cut reused the dither's 0.7 and 0.8 and Harsha found it dull
  next to the reference. At full strength the tiles are still only the
  ramp's own colours, so contrast against #343434 holds above 9:1.
  The field drifts along the bands over `u_time` (`MOSAIC_SPEED`, 2.5); its
  rAF loop yields to the trail's `tick()`, pauses while the tab is hidden or
  the band is scrolled away, and is still under prefers-reduced-motion. The
  ragged bottom edge is partly where the ramp turns white, and partly a
  hard floor: every tile below the first pill's top minus `MOSAIC_FLOOR_GAP`
  (20px) is forced to the page colour, edge scattered by `MOSAIC_FLOOR_RAG`.
  Harsha's rule: the mosaic stays inside the hero, above the pills. The
  floor is read from the DOM in `size()`, so it follows every breakpoint;
  the cursor lift can't pull colour below it either.

  **Cursor** (`MOSAIC_CURSOR`): a gaussian well of radius `MOSAIC_CURSOR_R`
  (150px), shaped by the field so it pools as a blob rather than a disc,
  that pushes tiles up the ramp by `MOSAIC_CURSOR_AMP` — so the tiles
  around the pointer go yellower, in the ramp's own colours — and, on top of
  that, makes each tile a richer version of itself: chroma scaled about its
  own luminance by `MOSAIC_CURSOR_CHROMA` (2.4), dimmed 7% so the yellow
  doesn't clip, and tilted warm by `MOSAIC_CURSOR_WARMTH` (0.3: red up by
  half, blue down by all — golden yellow, aqua cyan), blended in by
  `MOSAIC_CURSOR_VIVID`. Near enough the same hue that it reads
  as the gradient intensifying; white tiles have no chroma, so it dies out
  where the band does. Deepest tile stays above 7:1 against the copy. This
  was Harsha's ask after the ramp-lift alone: "slightly vibrant, enhances
  the gradient, doesn't disturb readability". The position
  is eased each frame and the strength eases in over the band and out when
  the pointer leaves, so the spot glides and fades. Harsha liked this
  version for being fluid and gooey and staying inside the hero. A coral
  accent colour was tried in its place on 2026-10-08 and rejected: it
  looked random against the gradient, and circular and big. Deliberately
  local — not the reference's long-press flood. Fine pointers only, for the
  same touch reasons as `CURSOR_PUSH`, which displaces the smooth ramp and
  stays off.
- **No grain animation on the dither.** A slow drift of the blue-noise tile
  across the ramp (colour static underneath) was built on 2026-10-06 and
  removed the same day — Harsha didn't like how it looked. The mosaic above
  is a different thing: the colour field moves, not the grain.
- **Cursor push** (`CURSOR_PUSH`, currently `false`). A decaying list of recent
  pointer positions, each shoving the wash outward and clearing to `--bg`.
  Paused at Harsha's request; the code is intact, flip the flag to restore it.
  It's additionally gated to fine pointers: on touch a tap fires `pointermove`,
  which punched a cleared circle wherever you tapped, and `pointerleave` never
  fires there, so the next tap drew a streak between the two.

Because the canvas now draws the band rather than only decorating it, it runs
on every device. If WebGL is missing it removes itself and the CSS
`.sky-gradient` shows through — undithered, but correct.

The stops live twice: as `--sky-tint` in CSS (fallback and button ink) and as
`RAMP` in the script (baked into a 256×1 lookup texture), and the phone's
set again as the phone `--sky-tint` and `RAMP_PHONE`. Change one, change
the other.

Because the band sits at `z-index: -1` under `body`, the page colour lives on
`html` and `body` is transparent — moving `--bg` back to `body` hides it.

The band and its canvas live inside `.sky-clip`, an absolute, overflow-hidden
box filling `body` (which is `position: relative` for it). The band is a
fixed 948px; unclipped, it stretched the document past the footer whenever
the list was short, leaving empty scroll room under it. The footer is a
fixed 49px and sits at the bottom of the page or of the screen, whichever
is lower.

The About button opens a folded letter (Figma node `967:595`): a 480 × 679
A4 sheet, square to the page — Harsha took the design's 4° tilt off — on the
same `rgb(42 42 42 / 0.4)` scrim as the
submit dialog. It's a `<dialog>` like that one, with no close control: Esc is
native and the backdrop click handler is shared. The exception is the phone
(below 600px): the sheet is taller than the screen, so the dialog becomes a
full-screen scroller with the letter inset 72px from the top and 56px from
the bottom (taps there hit the dialog itself and close it), plus a fixed
round close button in the top-right — Harsha found there was no way out
on mobile. The dialog's own focus ring
is suppressed — `showModal()` focuses the dialog box, which is invisible
behind the tilted sheet.

Measurements are the design's, expressed as fractions of the 480 width so a
smaller sheet keeps its proportions: 40px side margins, 32px head, creases at
236 and 443 (so the middle panel is the short one). Top-down: three polaroids
(Figma group `967:600`'s angles and centres), the greeting "Hey there, thanks
for stopping by." in Fraunces italic at weight 498, 24px, `SOFT 100`,
`WONK 0.36`, then 18px to 14px `#525252` Geist body on a 1.6 line height with
one empty line between paragraphs — Harsha's copy from 2026-10-18, three
paragraphs ending "Please feel free to share your feedback :)" — then "With
love," and the signature below it, near the right margin. Width is capped by
viewport height as well as width, so a short laptop screen gets a smaller
sheet. The script sets `--sheet-h` to width × √2, or the copy's natural
height if that's taller (narrow phones); below 600px the tilt goes, the
greeting may wrap and the dialog scrolls.

The photos are real polaroids, not the design's rounded placeholder cards —
Harsha asked for that: an 88px white card, 6px border with a 22px bottom, a
square print, 2px radius and a soft layered shadow. They fan into a straight
row on hover (`@media (hover: hover)`); without hover a tap toggles
`.is-spread` on all three copies of the group instead, and below 600px
`--card` shrinks on narrow phones so the fanned row fits inside the margins: rotation to 0, 12px gaps, 520ms
with a 40ms stagger. The hover is on the group, and hovering a child counts
as hovering the parent even where the child sits outside the parent's box,
so the spread cards keep the row open without the group being row-sized.

**The signature is static.** It's Harsha's signature from a photo (Figma
`978:14`, replacing an earlier one from `975:28`), as stroked SVG paths —
`signaturePaths` in the frontmatter, one path per pen stroke, in the order
Harsha marked on the photo in `978:11`. A live signing animation (dash
draw-on, power-law pacing) was built twice and dropped on 2026-10-18 —
Harsha found it didn't work; don't bring it back without asking.

The curves were made by placing rough guide points by hand, then
centring them on the ink: at every 3px, scan across the line, find the
ink's cross-section and take its midpoint (interpolating where a crossing
makes it ambiguous), and smooth the offsets. Hand-placed points alone
looked shaky; Harsha wants the lines as clean as the photo. The scripts
aren't in the repo; edit the path data directly to nudge a curve. The ink
is `--ink: #214E81`, sampled from the core of the photographed line;
stroke width 19 in an 861 × 993 viewBox, sized by height (92px) because
the signature runs on a steep diagonal. Three copies exist (one per
panel); only the first is exposed as an image ("Signed, Harsha").

It opens as a trifold. The sheet is rendered three times in the Astro
template (`letterFolds` in the frontmatter holds the crease positions as
fractions of the sheet height), one copy per panel. Each panel is a 3D hinge
with two faces: the front clips its copy of the sheet, shifted so the three
tile it; the back is plain paper pre-rotated 180°.

The paper has a pressed-fibre grain, after the paper in Plum's founders'
letter: `--paper-grain` is an SVG `feTurbulence` fractal noise lit with
`feDiffuseLighting` (so it reads as a surface, not speckle), squeezed by
`feComponentTransfer` into a narrow band near 1.0 and multiplied over
`#FCFCFC`; `--paper-mottle` adds faint warm, slightly stretched clouding.
Current: frequency 0.42, 2 octaves, surface 1.3, light at 68°, squeezed
to a 20% band; rendered mean ≈ 250, sd ≈ 1.5. This is the one Harsha
prefers. Tried and rejected: 0.72/3 octaves (read as stucco), and a
cold-press version after a Cressida note card (0.16–0.19, 4–5 octaves),
which was reverted on 2026-10-14 — don't reintroduce it without asking. Both are
generated data URIs, tileable via `stitchTiles`, no download. The grain is
on `.letter-sheet`, not the panel, so the three shifted copies line up and
the texture runs unbroken across the folds. Tune it against a render — the filter runs at device
resolution and comes out a few points darker on a 2× screen than in an
offscreen canvas. The faces hide their own
backfaces — putting `overflow: hidden` on the hinge flattens its 3D context
and the folded panel shows mirrored text, which is what the first version
did. Top panel swings down (origin bottom edge), then the bottom panel swings
up (origin top edge), with `translateZ` keeping them in front of the middle
the way a C-fold stacks. Panels start at ±170°, not 180°, so the overshoot
is hidden behind the fade-in; 760ms each, the top at 160ms and the bottom at
440ms. A quieter opening (a 10px rise and panels finishing from ±58°) was
tried on 2026-10-14 and reverted — Harsha prefers the full unfold.

There's one drop shadow, on `.letter`, not per panel: per-panel shadows let
the top panel's shadow land on the panel below at rest, which drew a hard
edge at the crease and made the letter read as stacked sheets. That shadow
is the size of the *open* sheet, so it fades in (`letter-shadow`, from
620ms) as the bottom panel comes down; shown from the start it outlined the
whole letter around the folded one — Harsha saw it as a bug.

Closing is animated, for both dialogs. A `<dialog>` closes instantly, so the
backdrop click and Esc (the `cancel` event, prevented) go through
`closeAnimated()` in the "Dialog close" section, which adds `.is-closing` —
a 180ms fade with a 6px drop, the scrim fading with it — and calls
`close()` on `animationend` for `dialog-out`, with a 400ms timeout as a
backstop. Reduced motion closes at once. The submit modal also opens with
the mirror of that (`dialog-in`: fade in rising 6px over 220ms); the letter
keeps its unfold. `dialog-in` must fill `backwards`, not `both`: a translate
animation left filled after it ends makes the dialog the containing block
for fixed children, which threw the Entry dropdown off by the dialog's
corner and clipped it — the dropdown looked dead. The creases are drawn as valley
folds lit from above, replacing the design's flat 4px `#F5F5F5` band at
Harsha's ask for a realistic fold: above each crease a long soft shade where
the paper slopes away from the light, then a dark hairline at the fold, then
a faint catch of light just below where it turns back up. Faces clip, so the
shade sits at the bottom of the panel above and the hairline and highlight
at the top of the panel below — and both halves meet at the same 2.6%
shade, so there's no brightness step at the fold line; the step is what read
as artificial. No hairline. Harsha wants it noticeable but never in the way
of reading; earlier, stronger cuts (and a tone shift on the middle panel)
were too obvious.
Everything animates off `dialog[open]`, so it replays on every open; all of
it is switched off under `prefers-reduced-motion`.

History: the first About was an envelope with the letter sliding out; it
became this folded letter on 2026-10-06 (Figma `937:24`, equal-thirds A4
with 10% margins, placeholder photos and a PNG signature wiped in with a
mask), and took its current design, photos and SVG signature on 2026-10-13.

Entry in the submit dialog is a multiselect listbox, not a text input: an entry
can be "Free, Paid". The trigger reuses `.form-input` so it matches the Social
Link field, plus a chevron; the panel lists `entryOptions` with ticks, and a
hidden input carries the joined `"Paid, Invite only"` string into the form. It
has no Escape-to-close handler — writing one tripped a false positive in the
environment's secrets scanner, which read the DOM property as a `.key` file.
Clicking outside closes it, and Escape still closes the dialog itself.

The submit dialog is built from Figma node `906:5559`: 590px wide, 24px/28px
padding, `#F8F8F8` card on a `rgb(42 42 42 / 0.4)` scrim — that scrim looks
green over the hero but is neutral; it was solved from the render against two
known backgrounds rather than guessed. It has no close control, matching the
design; Esc is native to `<dialog>` and a backdrop click dismisses it. Width is
capped (`min(590px, 100vw - 48px)`) rather than fixed, and the Entry/Social row
stacks below 420px, which the 1440-only frame doesn't cover. It opens and
closes with the short fade described under the About letter.

## Current state

- The submit form posts to a Google Apps Script web app — `SHEET_ENDPOINT` at
  the top of the inline script, wired up on 2026-10-06 — which appends a row
  to Harsha's "somewhere-in-chennai-submissions" sheet. Setup and gotchas are
  in `docs/submissions-setup.md`. Submissions are vetted by hand: nothing
  reaches the site automatically, `communities.js` stays the only source it
  reads. Setting the constant back to `""` turns the form off again.
- `README.md` is still the stock Astro starter template.
- The footer's "Last updated" date is resolved at build time in the page's
  frontmatter, from the last commit that touched `src/data/communities.js` —
  the community list, not the build, so restyling doesn't claim the directory
  changed. If that file has uncommitted edits it falls back to today, and if
  git is unavailable, to the file's mtime. Worth knowing: a host that does a
  shallow clone may have no history for that path, in which case mtime is
  checkout time — i.e. the build date.

## Conventions

- Section headers in both CSS and the inline script use `─` box-drawing rules;
  match that style when adding a section.
- Two-space indentation throughout.
- Links go to Instagram profiles; rows open them in a new tab with `noopener`.
