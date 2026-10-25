# LightTurtles slide toolkit

Write decks in Markdown, build them with pandoc into **one self-contained HTML file** (reveal.js), and export a **PDF** with one page per build step.

**[CHEATSHEET.md](CHEATSHEET.md)** has every layout recipe and feature, with copy-paste examples.

## Setup (once)

```sh
sudo apt install pandoc scons                 # pandoc ≥ 3.0
cd slides-kit && npm install                  # reveal.js + decktape (PDF export)
ln -s "$PWD/slides" ~/.local/bin/slides       # put the `slides` command on your PATH
```

## Daily use

The kit holds only tools. Your talks live in their own work folders, and the outputs land next to the source:

```
slides-kit/          ← this repo: theme, scripts, build rules
work/conf2026/
├── example.md       ← you write this (and drop images beside it)
├── mcu.svg
├── example.html     ← built: one self-contained file, share it as-is
└── example.pdf      ← built: handout / backup
```

From inside a work folder:

```sh
slides new=example          # creates example.md here from the kit's skeleton.md
slides watch=example        # rebuild on every save; keep the HTML open and refresh
slides                      # build every deck in this folder → <name>.html
slides example              # just one deck
slides example.pdf          # PDF (`slides pdf` for every deck here; HIDDEN=show includes hidden slides)
slides -c                   # remove this folder's built .html/.pdf
slides -h                   # all commands and options
```

A **deck** is any `.md` in the folder that starts with `---` front matter, so a plain README or notes file is left alone. Several decks can share one folder.

The HTML embeds every image, font and script, so `example.html` alone is everything someone needs to view the talk later, offline, on any computer with a browser.

In the browser: **→/Space/scroll down** next, **←/scroll up** back (every slide and build step, in order), **S** speaker view (notes + timer + next slide), **Shift+H** show/hide hidden slides, **F** fullscreen, **O** overview, **B** black screen, **?** all shortcuts.

If a widget on a slide has the keyboard, it shows an orange focus ring and arrows go to it; **Esc** gives them back to the deck. Mouse use of a slider or button returns focus automatically, and changing slides always does.

## Layouts

All layouts share the brand (colors, IBM Plex fonts, turtle, code theme); they differ in where the title, logo, accents and footer go. Pick one per deck in its front matter:

```yaml
layout: notebook     # notebook | rail | banner | frame
```

| Layout     | Look |
|------------|------|
| `notebook` | Default. Titles top-left on a warm-gray rule with a short green accent; green edge bar and wordmark on the title slide. |
| `rail`     | Green rail down the left of every slide with the turtle at its foot; the title slide widens it into a panel; section breaks go full green. |
| `banner`   | Each slide title sits in a full-width green header band with the turtle and name; all-green title slide with the full logo. |
| `frame`    | Echoes the logo: every slide inside a charcoal rounded rectangle; the title slide is laid out like `logo-full`. |

To try a deck in another layout without editing it: `slides LAYOUT=banner`. (Builds that way overwrite the deck's `.html`; run plain `slides` to go back.)

To add a layout, drop a new `theme/layouts/<name>.css` in place; the base styles in `theme/lightturtles.css` are already applied, so a layout only needs its own placements.

## Writing slides

| You want                     | Write                                                     |
|------------------------------|-----------------------------------------------------------|
| A slide                      | `## Title`                                                |
| A section divider            | `# Section name`                                          |
| Group slides, no divider     | `# Section name {.hidden}` (or just `# {.hidden}`)        |
| A hidden/backup slide        | `## Title {.hidden}`; **Shift+H** shows/hides them        |
| Speaker notes                | `::: notes` … `:::`                                       |
| Bullets one click at a time  | wrap the list in `::: incremental` … `:::`                |
| Any element as a build step  | `[text]{.fragment}` or `::: fragment`                     |
| Two columns                  | `:::: columns` / `::: {.column width="40%"}`              |
| A callout box                | `::: callout`                                             |
| A big one-liner slide        | `## The idea {.statement}`                                |
| Hide the footer on one slide | `## Title {.no-footer}`                                   |
| Callout colors               | `::: callout` (orange), `{.callout .green}`, `.teal`, `.bronze` |
| Slide background             | `## Title {data-background-color="#046307"}` or `data-background-image="photo.jpg"` |
| Precise layout               | any raw HTML block — Markdown resumes after it            |
| An interactive widget        | raw `<div>` + `<script>` + `<style>` in the slide         |
| Zoom/fly around a graphic    | see below                                                 |

### Zoom tour

```markdown
## Inside a microcontroller {.zoomtour}

![](mcu.svg)

[Caption for the first stop]{.zt-stop view="100 140 440 320"}
```

`view="x y w h"` is the rectangle of the graphic to show: top-left corner, then width and height, in the SVG's `viewBox` units (pixels for a PNG/JPG); `view="all"` is the whole graphic. Put a `view` on the slide heading too (`{.zoomtour view="…"}`) to make the slide open there. To get the numbers, open the built deck, go to the slide, press **Z**, and drag boxes around each region; the picker hands you the lines to paste. Details in the cheatsheet.

## Where things live

```
defaults/lightturtles.yaml   pandoc options (slide size, transitions, numbering)
theme/lightturtles.css       brand tokens (from brand.css), fonts, shared components
theme/layouts/*.css          one file per layout: title, accents, footer placement
theme/brand/                 turtle.svg (+ paper/ghost variants), logo-full/-wordmark/-icon.svg
theme/fonts/                 IBM Plex Serif + Mono (woff2, OFL licence), embedded in every deck
theme/code.theme             code-highlighting colors (pandoc JSON theme, brand palette)
js/zoomtour.html             zoom-tour behaviour
js/chrome.html               per-slide footer: turtle, name, deck title, slide number
js/hidden.html               {.hidden} slides: skipped in the talk and PDF; Shift+H toggles
js/focus.html                keeps arrow keys driving the deck around in-slide widgets
js/wheel.html                scroll wheel steps forward/back, one step per gesture
templates/revealjs.html      pandoc's page template, plus phone fixes (no scroll view, pinch-zoom, touch arrows)
slides                       the command you run from a work folder (wrapper around SCons)
SConstruct                   build rules
skeleton.md                  template for `slides new=…`
```

## Notes

- **Portable means offline.** `--embed-resources` inlines everything at build time, so the HTML works with no network. That's why the fonts are vendored `.woff2` files rather than Google Fonts links.
- **Logos** are all vector, in `theme/brand/`. `turtle.svg` is the exact turtle outline from the Tk EPS files. To replace a logo, convert its EPS with `tools/eps2svg.sh logo.eps` (Inkscape, text outlined) and drop the SVG into `theme/brand/` under the same name.
- **Math:** not wired up yet (pandoc's default MathJax fetches from a CDN). Easy to add with a vendored KaTeX when needed.
- **Phones:** the deck behaves as on a laptop (swipe or touch arrows, pinch-zoom). reveal.js's automatic "scroll view" for narrow screens is turned off in `templates/revealjs.html`, because it rebuilds the deck on rotation and breaks footers, hidden slides and zoom tours.
- **PDF:** decktape drives a headless Chrome. Use `slides talk.pdf CHROME=/usr/bin/chromium` to reuse an installed browser.
- **Rebuilds** are content-based: a deck rebuilds when its Markdown, any other file in its folder (images, data, subfolders), or any kit theme/script/defaults file actually changes. If two decks share a folder, changing an image rebuilds both.
- **Without the wrapper:** `cd slides-kit && scons WORK=../work/conf2026 example` does the same thing. SCons's cache (`.sconsign.dblite`) stays in the kit and is git-ignored, so nothing but the outputs is ever written to a work folder.
