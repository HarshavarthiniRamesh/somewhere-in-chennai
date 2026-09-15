# Somewhere in Chennai

A single-page Astro site: a curated, filterable table of Chennai communities and
meetups. Designed by Harsha, built with Claude Code.

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

### `src/data/communities.js`

Each entry is an object with `name`, `description`, `categories` (an array),
`entry`, `frequency`, and `link`. A row with an empty `link` renders as inert —
not clickable, no focus ring — so an entry can be added before its link is known.

`filterCategories` is exported separately and its order matches the Figma
design. It is not derived from the entries, so a new category has to be added
in both places to appear as a pill.

### `src/pages/index.astro`

The inline script is organised in commented sections: filter pills, column sort,
row click, About envelope, submit modal, submit form. `applyFilter()` is shared —
it both hides non-matching rows and renumbers the visible ones, so it must be
called after anything that reorders or re-shows rows.

Row numbers are display-only and always recomputed; don't treat them as IDs.

### `src/styles/global.css`

Colors, fonts and layout widths are CSS custom properties on `:root` — use the
tokens (`var(--text-muted)`, `var(--col-name)`) rather than hardcoding values.

Three typefaces, all self-hosted via `@fontsource-variable`: Geist (sans, body),
Geist Mono (labels, numerals), Fraunces (display italic, accents).

`.sky-gradient` is currently disabled with `display: none` — an earlier
background treatment kept around deliberately. Leave it unless asked.

## Current state

- The submit form's `SHEET_ENDPOINT` (top of the inline script) is an empty
  string, so submissions only show an "aren't wired up yet" message. Setting it
  to a Google Apps Script URL activates the existing POST.
- `README.md` is still the stock Astro starter template.
- The footer's "Last updated" date is hardcoded and edited by hand.

## Conventions

- Section headers in both CSS and the inline script use `─` box-drawing rules;
  match that style when adding a section.
- Two-space indentation throughout.
- Links go to Instagram profiles; rows open them in a new tab with `noopener`.
