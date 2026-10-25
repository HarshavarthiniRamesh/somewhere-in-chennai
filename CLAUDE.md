# Somewhere in Chennai

A single-page Astro site: a curated, filterable table of Chennai communities and
meetups. Designed by Harsha, built with Claude Code. "Somewhere in Chennai" is
the project's name; the site's own title is **"Not home. Not work. / But the
secret third thing."** (two lines, hard break, Figma `956:142`) since
2026-10-10, after a day as "Chennai Communities Guide". The tagline is "A
directory of communities and activities in Chennai." since 2026-10-22.

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
decision, not just an alphabetical one. (On a phone the pills scroll in
one row, so there the order is just reading order.)

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

Both nav buttons are cream-outlined; Submit is filled cream, and About fills
on hover.

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

Header labels are `--text-muted`, a step back from the cream rows.

Keyboard: the sort headers are `div`s with `role="button"`, `tabindex="0"`
and Enter/Space wired to their click. Rows with a link are Tab stops
(Enter opens them). Both get a 1.5px cream ring inside the box on
`:focus-visible` only, and a focused row takes the hover state too.

A hover photo card for each row (a tilted photo from the community's
Instagram, above the hovered row) is built and parked on the local branch
`row-photo-preview` — Harsha is holding it until every community has
agreed to have a photo shown.

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
failure. The size is `min(48px, ceiling)` on desktop and tablet: 48px is the
design size (Figma `1063:4159`, 2026-10-23 — it was 64px; 18px to the
tagline; line height 1.08, or 1.1 on phones), and the
ceiling is the widest the longer line can be and still fit on one line with
5% to spare, so no width can wrap it. Ceilings are solved from the line's
width-to-font-size ratio, **11.22 for "But the secret third thing."**
(measured in the browser with the real font, opsz auto):
`0.95 × (vw − gutters − anything else on the row) / 11.22`. That gives
`8.47vw - 4.07px` on the phone (24px gutters), `8.47vw - 6.78px` on the
tablet (40px), and `8.47vw - 19.31px` on the desktop (56px gutters plus the
92px nav and a 24px gap beside it). The 48px cap is the smaller term from 795px
on the desktop row and 647px on the tablet; below that the ceiling rules —
44px at 600, 33px at 440, 23px at 320 (the phone has no cap, just the
ceiling). Re-measure the ratio and re-solve all
three if the title text, the font or `--page-pad` ever change.

The title sets only `SOFT` and `WONK` and lets `opsz` follow the size, as the
Figma text does; the modal title still pins `opsz 100`. It carries no
optical margin: measured from pixels, the italic N and B start within half a
pixel of the tagline's A. (The "Chennai…" title needed `-0.057em` for its C.)

Tagline is `clamp(16px, calc(0.5747vw + 11.72px), 18px)` at every size — 18px on
the desktop (it was 20 until 2026-10-23), 16px from 744 down — except below 600px, where it's set to 15px.
On a phone the pills start 160px below the buttons (Figma `986:19`).
On desktop they sit at y 408 as in Figma `1063:3597` (203px under the
hero).

Tablet stacks the header (title, tagline, then both buttons) but keeps the
desktop table and lets it scroll sideways. Only the phone restacks it. 1120px is
where the title, the nav and the page padding stopped fitting on one row
with the original string (the longer title shrinks there instead; see above).

The phone table is a different layout, not the desktop table shrunk: rows
become a grid of `56px 1fr` with the number spanning three stacked lines —
name, description, then a `.cell-meta` line joining category, entry and
frequency as `A, B / Free / Monthly`. `.cell-meta` is built in the markup and
hidden above 600px; `.cell-category`, `.cell-entry` and `.cell-freq` are hidden
below it, and the table drops back to `display: block` so nothing scrolls
sideways on a phone.
Spacing and type follow Figma `987:24`: 18px top and bottom, name 16px
medium, 6px, description 14px on 1.4 (two lines, then an ellipsis; rows grow
with it — reserving two lines for every row was tried on 2026-10-18 and
dropped), 10px, meta 14px in `#747474`.

On a phone the filter pills are one row that scrolls sideways (Figma
`987:89`, 6px gaps) rather than wrapping into five rows. The scroller runs
edge to edge with the gutter as padding, scrollbar hidden; the pill cut off
at the right edge is the cue that there's more. A tapped pill that's partly
off screen is slid into view (`revealPill`, which scrolls the row only,
not the page).

**Colour and page art** (Figma `1016:14`, 2026-10-22): an olive page
(`--bg: #3D4013`), cream type (`--text: #DFD6C2`), tan dashed rules and pill
outlines (`--line: #908262`), and `--text-muted: #B2AC91` (cream at 72% over
the olive, 4.7:1) for secondary text. Row and pill hover are both the noise's olive, `#8A903A`, at 6% (Harsha wanted it very faint)
(`--row-hover`, `--pill-hover`); cream at 6% read as a white film. Translucent
on purpose: a solid
`#3A3D14` was tried and hid the branch behind a hovered row, so the row
looked cut out of the art. The table and footer
rules aren't CSS dashed borders: Figma's dashes are 2px on, 4px off, sparser
than the browser's, so they're a repeating gradient (`--rule`, `--rule-top`)
measured off Figma's render. Pill outlines
match: each pill has an inline `.filter-dash` SVG whose rect is stroked
`2 4` round the 8px corners, on the line a 1px border would take (the
real border is transparent and only holds the size). The dialogs re-theme themselves by
resetting those custom properties, which re-themes everything inside them.
The letter is white paper and keeps the original dark-on-white set. The
submit card is the page's own material: the olive with 4% cream mixed in
(`#434619`), a touch lighter and less saturated than the page, 8px
corners, separated only by a soft shadow, on a `rgb(20 21 6 / 0.55)` scrim
(0.3 left the page competing with the card). Cream type, a cream Submit button (the only bright thing in the
card), and its own copy of the grain (the page's can't reach the top
layer). Fields are plain inputs: a solid outline in cream at 20%, 55% on
focus. Former greys are cream mixed into the olive: subtitle 90%, labels
80%, placeholders 50%. Tried and dropped on 2026-10-22: an inverted beige
card with green type (a flat slab); a lighter `#45491A` card on a dark scrim
(it glowed and looked pasted on); a tan outline round the card, and the
page's dashed outlines on the fields — Harsha: inputs are never dashed in
real life. Keep the dashes to the table, pills and footer.

`.page-art` is a fixed, screen-sized layer behind the content, carrying the
page colour. The page is flat olive under the grain, so a fixed olive layer
looks identical — and being self-contained, it lets the browser scroll the
page over the blended branch without re-blending every frame. (With only
the branch fixed, blending against the scrolling page, it lagged and
shifted on a phone.) It also has to carry the colour for the blend: it's
its own stacking context, and the blend needs a backdrop inside it.

- **The flowering branch** (`public/art/flower.webp`), a greyscale print
  with `mix-blend-mode: multiply` at 16% opacity (Figma `1039:1740`,
  2026-10-23): a faint darker shadow in the page's own green. It replaced a
  colour print with `mix-blend-mode: hue`, which turned the olive red-brown
  where it lay. It stays at the top of the screen while the page
  scrolls under it. Placement is per breakpoint, from Figma, as
  `--flower-top/right/w/rot` (centre from the top and right edge): desktop
  `1016:587` (71, 478, 578.75px of the 1440 frame, −73.11°, written as `vw`
  so it scales with wide screens — fixed px left it tiny in the corner at
  4K), tablet the same at 80%, phone
  `1039:1848` (73.4, 195.8, 213.5px, −77.82°).
- **Noise over the whole page**, as `body::after`, absolute and above the
  content (z-index 1, no pointer events) as Figma's noise frame is — so
  solid hover fills keep their grain. Figma's
  noise frame at 8%, made with the Noise & Texture plugin (Duo tone,
  `#8A903A`, 1px, 79% density). `public/art/noise.png` is a 512px crop of
  the exported 1024px tile, cut to a 64-colour palette (~174KB) and drawn
  1:1 so each dot is 1px. (Figma's own canvas stretches the tile to the
  1440 frame, so its dots render at 1.4px.)

`body`
must not have `height: 100%` — it did once, which capped it (and so the
noise) at one screen while the content overflowed it; `min-height: 100dvh`
alone keeps the footer down. Because
the layer sits at `z-index: -1` under `body`, the page colour lives on
`html` and `body` is transparent. The footer is a fixed 49px and sits at the
bottom of the page or of the screen, whichever is lower.

This replaced the yellow-to-blue hero gradient: a WebGL band with Figma's
blue-noise dither, a parked "mosaic" mode and cursor push, a phone-specific
ramp, and the Submit label as a window onto the wash. All of that was
removed on 2026-10-22; it's in git history at commit `23041ba` if it's ever
wanted back.

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
Harsha asked for that: an 88px card (warm white `#FEFCF7` under the paper
grain at 150px, so it reads as matte card stock — plain `#fff` looked like a
sticker on the creamy paper), 6px border with a 22px bottom, a
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
the signature runs on a steep diagonal. As drawn it rose at ~44°; Harsha
found it too slanted, so the SVG is turned `rotate: 25deg` (to ~19°) rather
than the path data being redrawn. Three copies exist (one per
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
`--paper`, a creamy white `#FBF8F0` since 2026-10-22 (it was `#FCFCFC`;
the warmer one reads as real paper against the olive page); `--paper-mottle` adds faint warm, slightly stretched clouding.
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
padding (the card was `#F8F8F8` on a `rgb(42 42 42 / 0.4)` scrim until
2026-10-22) — neutral; it was
solved from the old render against two known backgrounds rather than
guessed. It has no close control, matching the
design; Esc is native to `<dialog>` and a backdrop click dismisses it. Width is
capped (`min(590px, 100vw - 48px)`) rather than fixed, and the Entry/Social row
stacks below 420px, which the 1440-only frame doesn't cover. It opens and
closes with the short fade described under the About letter.

## Favicon

A five-petal cherry blossom drawn as SVG paths (2026-10-25), petals
notched at the tips, with a tan `#908262` centre. Just the flower, on a
transparent background: Harsha found the first version's olive tile read
as a ring with an odd shadow in a dark tab strip. The petals follow the
browser theme via a `prefers-color-scheme` rule inside the SVG — olive
`#3D4013` on light tabs, cream `#DFD6C2` on dark. A real emoji was the
ask, but one can't be recoloured to the palette and differs per platform,
so it's drawn. `favicon.ico` (16/32/48, olive petals, for browsers without
SVG icons) is rasterised from the SVG; `apple-touch-icon.png` keeps the
olive tile, because iOS fills a transparent home-screen icon with black.
Regenerate them if the SVG changes, and bump the `?v=` on the three icon
links in the head: browsers cache favicons hard, and a new URL is the only
reliable way to make them refetch.

## Share preview

`public/og.jpg` is the link-preview image (`og:image`, Twitter
`summary_large_image`): the page as it first loads in a 1200 × 630 window,
captured at 2× in headless Chrome and scaled down, dev toolbar removed.
It's a screenshot, so it goes stale when the hero, pills or palette
change — recapture it then. A hero-only version (pills and table hidden)
was tried and was mostly empty olive. The tags need absolute URLs, built
from `site` in `astro.config.mjs` (`https://nothome-notwork.vercel.app`);
change that if the site lives elsewhere.

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
