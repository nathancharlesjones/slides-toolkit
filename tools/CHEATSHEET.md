# Slides cheatsheet

Everything here works with this kit's build (`slides`), and was checked in a real browser. Recipes for the common cases come first; the full catalogue of what's possible is further down.

**Contents**

1. [Commands](#commands)
2. [Deck anatomy](#deck-anatomy)
   - [Sections and hidden slides](#sections-and-hidden-slides)
3. [Layout recipes](#layout-recipes)
4. [Builds and motion](#builds-and-motion)
5. [Backgrounds](#backgrounds)
6. [Zoom tours](#zoom-tours)
7. [Interactive widgets](#interactive-widgets)
8. [Presenting and sharing](#presenting-and-sharing)
9. [Class reference](#class-reference)
10. [Limits and gotchas](#limits-and-gotchas)
11. [Customizing the kit](#customizing-the-kit)

---

## Commands

Run from the work folder that holds the deck:

| Command | Does |
|---|---|
| `slides new=talk` | create `talk.md` from the skeleton |
| `slides watch=talk` | rebuild `talk.html` on every save; refresh the browser to see it |
| `slides` / `slides talk` | build every deck here / just one |
| `slides talk.pdf` | PDF, one page per click |
| `slides talk.pdf HIDDEN=show` | PDF including `{.hidden}` slides |
| `slides LAYOUT=rail` | preview the deck(s) in another layout (overwrites the .html) |
| `slides -c` | delete this folder's built .html/.pdf |
| `slides -h` | every command and option |

---

## Deck anatomy

```markdown
---
title: How Hardware Gets Hacked
subtitle: An introduction for firmware engineers
author: Nathan Jones
date: November 2026
layout: notebook          # notebook | rail | banner | frame
---

# Part one                ← "#" = section-divider slide

## First slide            ← "##" = a new slide

Body text, lists, images…

::: notes
Only you see this, in the speaker view (press S).
:::

## Second slide {.statement}     ← classes/attributes on the heading apply to the whole slide
```

- The title slide is generated from the front matter; you don't write it.
- A slide doesn't need a visible title: `## {.no-footer}` with nothing after `##` makes an untitled slide.
- `---` on its own line also starts a new slide (untitled).

### Sections and hidden slides

Structure and display are separate: `#` groups slides, and `{.hidden}` keeps a slide from being shown.

```markdown
## Intro                      ← slides before the first "#" stand on their own
# Escape hatches              ← divider slide; groups the "##" slides after it…
## Raw HTML
## Widgets
# Wrap-up {.hidden}           ← …until the next "#". Hidden: no divider shown, but
## Backgrounds                     Backgrounds and Questions? form their own group
## Questions? {.statement}
## Backup: timing data {.hidden}   ← a backup slide for Q&A
```

- `# {.hidden}` (no name) works too, when you just want a group boundary.
- **Hidden slides** are skipped when presenting, left out of the overview and the PDF, and not counted in the slide numbers.
- **Shift+H** shows them in place (tagged HIDDEN), in the main window, the overview, and the speaker view; press it again to put them away. Good for rehearsal and for pulling up a backup slide in Q&A.
- `slides talk.pdf HIDDEN=show` makes a PDF that includes them (a speaker's copy). Rebuild without the option for the audience copy.
- Grouping shows in the overview (**O**) and in the URL, not in normal navigation: → always steps through every visible slide in order.
- Hidden isn't secret: the slides are still in the .html file, and anyone can press Shift+H.

---

## Layout recipes

The slide is **1920 × 1080**. Content sits inside the layout's padding (about 110 px each side).

### Text

```markdown
**Bold** is green · *italic* · `inline code` · [link](https://…)
[Highlighted]{.mark} · [hot orange]{.hot} · [muted]{.muted} · [monospace]{.mono}

- Bullets          1. Numbered
- nest by              1. nest by
    - indenting 4          indenting 4

> A quote, set in walnut italics with a bronze bar.
```

Smaller text for a dense slide: wrap it in `::: small` … `:::`.

### Two columns

```markdown
:::: columns
::: column
Left side
:::
::: column
Right side
:::
::::
```

- **Uneven:** `::: {.column width="35%"}` and `::: {.column width="65%"}`.
- **Three or more:** just add more `::: column` blocks.
- **Vertically centred** (e.g. text beside an image): `:::: {.columns align=center}` (also `align=bottom`).
- Columns can hold anything: images, code, callouts, lists.

### Text beside an image

```markdown
:::: {.columns align=center}
::: {.column width="40%"}
- Point one
- Point two
:::
::: {.column width="60%"}
![](board.jpg)
:::
::::
```

### Images

```markdown
![Optional caption](board.jpg)                 ← caption appears below
![](board.jpg){width=50%}                       ← or height=400px
![](board.jpg){.r-stretch}                      ← fill the slide's remaining height
```

A full-bleed photo is a background, not an image: see [Backgrounds](#backgrounds).

### Callouts and statement slides

```markdown
::: callout
Orange bar (default). Variants: {.callout .green}, .teal, .bronze
:::

## One idea per slide. {.statement}            ← big, vertically centred, no body needed
```

### Code

````markdown
```c
int main(void) { return 0; }
```
````

Highlighted in the brand palette. Long lines scroll sideways rather than wrapping, so keep lines short (about 60 characters fits a two-column layout).

### Tables

```markdown
| Part     | Cost |
|:---------|-----:|          ← colons set left/right alignment
| MCU      | 1.20 |
```

### Footnotes

```markdown
Glitching works on most parts.[^1]

[^1]: O'Flynn, "Fault Injection", 2016.
```

The note appears at the bottom of the same slide.

### Precise placement: raw HTML

Any HTML works anywhere, and Markdown resumes right after it. Coordinates are slide pixels (1920 × 1080):

```markdown
<div style="position:absolute; left:1300px; top:300px; width:500px">
  Pinned here, regardless of the rest of the slide.
</div>

<div style="display:grid; grid-template-columns:repeat(3,1fr); gap:40px">
  <div class="callout">One</div>
  <div class="callout green">Two</div>
  <div class="callout teal">Three</div>
</div>
```

Inside raw HTML, use HTML (`<b>`, `<img src="x.png">`); Markdown isn't parsed there. Colors are available as CSS variables: `var(--lt-primary)`, `--lt-teal`, `--lt-orange`, `--lt-bronze`, `--lt-walnut`, `--lt-ink`, `--lt-paper`, `--lt-line`, `--lt-muted`.

### reveal.js layout helpers

| Class | Effect |
|---|---|
| `::: r-stack` | children sit on top of each other; with `.fragment` on each, they replace one another per click |
| `::: r-hstack` / `::: r-vstack` | children in a centred row / column |
| `{.r-stretch}` on an image | fill the remaining slide height |
| `::: r-fit-text` | text sized to fill the slide *width* (good for one long line, not one short word) |

---

## Builds and motion

### Reveal on click

```markdown
::: incremental            ← every list item inside appears one click at a time
- One
- Two
:::

[Appears on a click]{.fragment}

::: fragment
A whole block on one click.
:::
```

### Fragment styles

Add one of these after `.fragment`: `[text]{.fragment .fade-up}`

| Appear | Change after appearing | Disappear |
|---|---|---|
| (default) fade in | `highlight-red` / `highlight-green` / `highlight-blue` (text color) | `fade-out` |
| `fade-up` `fade-down` `fade-left` `fade-right` | `highlight-current-red` (only while current) | `semi-fade-out` |
| `grow` `shrink` | `strike` | `fade-in-then-out` |
| `zoom-in` | | `fade-in-then-semi-out`, `current-visible` |

### Fragment order

Fragments run top to bottom. To change that, number them: `[shown second]{.fragment fragment-index=2}`, `[shown first]{.fragment fragment-index=1}`. Fragments with the same number appear together.

### Replace images in place

```markdown
::: r-stack
![](step1.png){.fragment}

![](step2.png){.fragment}
:::
```

### Auto-animate (morph between slides)

Give two consecutive slides `data-auto-animate=""`. Elements that match between them (same text, or the same `data-id`) glide to their new position, size, and color:

```markdown
## Before {data-auto-animate=""}
[Buffer]{data-id="buf" style="display:inline-block; padding:20px; background:var(--lt-primary); color:white"}

## After {data-auto-animate=""}
[Buffer]{data-id="buf" style="display:inline-block; padding:60px; margin-left:900px; background:var(--lt-orange); color:white"}
```

### Transitions

Deck default is a quick fade. Per slide: `## Title {data-transition="zoom"}` (also `none`, `slide`, `convex`, `concave`).

---

## Backgrounds

```markdown
## Title {data-background-color="#046307"}                     ← text flips to light automatically
## Title {data-background-image="photo.jpg" .dark}             ← .dark = light text (needed for photos)
## Title {data-background-image="photo.jpg" data-background-size="contain"}
## Title {data-background-gradient="linear-gradient(#046307, #2f7f73)" .dark}
```

- Text over a busy photo: `[Readable caption]{.scrim}` (or a `::: scrim` block) puts it on a dark panel.
- Background images are embedded in the HTML like everything else.
- **Video:** use an inline video, which *is* embedded:
  `![](demo.mp4){width=70% data-autoplay="" loop="" muted=""}`.
  Avoid `data-background-video`: pandoc doesn't embed it, so it breaks when you share the HTML. Videos make the file big; trim them first.

---

## Zoom tours

One big graphic; each click flies the camera to the next region, with a caption.

```markdown
## Inside a microcontroller {.zoomtour}

![](mcu.svg)

[The CPU: fetch, decode, execute.]{.zt-stop view="100 140 440 320"}
[Flash and SRAM.]{.zt-stop view="700 140 380 430"}
[]{.zt-stop view="1090 140 420 660"}          ← empty brackets = no caption
```

### What the four numbers mean

`view="x y w h"` is a **rectangle on the graphic** that you want the screen to show:

- `x y`: its **top-left corner**, measured from the graphic's top-left; y counts **down**.
- `w h`: its **width and height**.
- **Units are the graphic's own:** for an SVG, the units of its `viewBox` (the numbers in `<svg viewBox="0 0 1600 900">`); for a PNG/JPG, pixels.

The camera centres that rectangle and scales it to fill 80% of the screen, leaving room for the title tag and caption. The rectangle can be any shape; a tall one just gets more room at the sides.

`view="all"` means the whole graphic (nearly edge to edge).

### Where the slide opens, and zooming out

Before the first click a zoom-tour slide shows the whole graphic, unless you give the **slide itself** a view. Then it opens right there, with no flight in, and a stop can zoom *out*:

```markdown
## Memory {.zoomtour view="720 150 340 190"}     ← opens zoomed in on Flash

![](mcu.svg)

[Flash and SRAM sit side by side…]{.zt-stop view="700 140 380 430"}
[…on one chip.]{.zt-stop view="all"}          ← zoom out to the whole graphic
```

**Zoom-first talks:** several zoom-tour slides can share one graphic, each opening at its own view, with ordinary slides in between. Arriving at a zoom-tour slide (forward or back) puts you at its view at once; only moves between stops on the same slide fly. Each slide carries its own copy of the graphic, so a large image repeated on many slides makes the HTML bigger.

### Getting the numbers: the picker (recommended)

1. Write the slide with the image and no stops (or placeholder stops), build, and open the HTML.
2. Go to the zoom-tour slide and press **Z**.
3. **Drag a box** around the first region. The camera flies there *exactly as it will in the talk*. Not quite right? Drag a new box; you can draw on the zoomed-in view to refine.
   Or navigate by hand: **scroll** to zoom at the cursor, **Shift-drag** to pan, then press **Enter** to make a stop from exactly what's on screen.
4. Repeat for each stop. The panel lists them as ready-to-paste lines. **Copy all**, paste into the Markdown, and replace the `Caption N` text.
5. **Backspace** undoes the last stop, **H** returns to the slide's starting view, **V** hides the panel, **Z** or **Esc** exits.

Pick mode is only an authoring aid; nothing about it shows in the talk unless someone presses Z.

### Getting the numbers: Inkscape

Draw a rectangle over the region and read **X, Y, W, H** from the toolbar. These equal the `viewBox` units only when the document's *scale* is 1 (File → Document Properties → Scale = 1.0, and the toolbar's unit set to px). With any other scale, use the picker.

### Tips

- **SVG** stays razor-sharp at any zoom; a photo gets soft as you zoom in, so use a high-resolution one.
- Make text in the graphic large enough to read *at the zoom level you'll show it*, not at full view.
- The PDF export gets one page per stop.

---

## Interactive widgets

A slide can hold any HTML, CSS, and JavaScript: sliders, canvases, simulators, quizzes.

```markdown
## Voltage divider

<div class="divider-demo">
  R2: <input type="range" min="1" max="100" value="50"> <output></output>
</div>

<script>
document.querySelectorAll('.divider-demo').forEach(el => {
  const r = el.querySelector('input'), out = el.querySelector('output');
  const draw = () => out.textContent = (3.3 * r.value / (10 + +r.value)).toFixed(2) + ' V';
  r.addEventListener('input', draw); draw();
});
</script>

<style>
.divider-demo input { width: 600px; accent-color: var(--lt-primary); }
</style>
```

- **Scope everything to a class** (`.divider-demo`), not an `id`, and loop with `querySelectorAll`. That way two copies on different slides, or two decks pasted together, don't collide.
- Use the brand variables (`var(--lt-primary)`…) so widgets match every layout.
- Scripts run once, when the deck loads. To react when the slide appears, use `Reveal.on('slidechanged', e => { if (e.currentSlide.contains(el)) … })`.
- **Keyboard:** while a widget control has focus (orange ring), the arrow keys go to it. **Esc** hands them back to the deck; mouse use of sliders and buttons returns focus automatically.
- **Scroll wheel:** over a widget's own controls or anything that scrolls, the wheel works on that element, not the deck. To keep the wheel off the deck over a whole widget (say, a canvas you zoom with the wheel), give its container the class `no-wheel`.
- In the PDF, widgets appear as they look when the deck first loads; they're live only in the HTML.

---

## Presenting and sharing

| Key | Does |
|---|---|
| **→ / Space / clicker / scroll down** | next step; **← / scroll up** back |
| **S** | speaker view: notes, timer, next slide (allow the pop-up) |
| **Shift+H** | show / hide hidden slides |
| **F** | fullscreen |
| **O** or **Esc** | overview of all slides |
| **B** or **.** | black screen |
| **Z** | zoom-tour picker (on zoom-tour slides) |
| **?** | all shortcuts |

- **Share the `.html`**: one file holding every image, font, and script. It opens offline in any browser.
- **The `.pdf`** is a handout or backup: one page per click, widgets frozen, links still clickable.
- The URL tracks your slide (`…#/my-slide`), so reloading after a rebuild keeps your place.
- **On phones and tablets** the deck shows exactly as on a laptop: swipe or tap the on-screen arrows (shown on touch screens only) to step through; pinch to zoom in on detail. In portrait the slide is letterboxed, so suggest rotating, or point phone readers at the PDF.
- Some phone browsers (Samsung Internet, Chrome with its "dark theme for sites" setting) repaint pages dark. Decks follow that setting; colors in the graphics will be darkened too.

---

## Class reference

**On a slide heading** (`## Title {.class}`):

| Class / attribute | Effect |
|---|---|
| `.statement` | one huge centred line |
| `.zoomtour` | zoom-tour slide |
| `.no-footer` | hide the footer |
| `.hidden` | skip this slide (on `#`: group without a divider); Shift+H shows it |
| `.dark` | light text (for dark photos/gradients) |
| `data-background-color/-image/-gradient/-size` | backgrounds |
| `data-transition` | per-slide transition |
| `data-auto-animate=""` | morph from the previous slide |

**On a block** (`::: class`) **or span** (`[text]{.class}`):

| Class | Effect |
|---|---|
| `callout` (+ `green` / `teal` / `bronze`) | panel with a colored bar |
| `columns` / `column` | column layout |
| `incremental` / `nonincremental` | list items one click at a time / all at once |
| `fragment` (+ style) | appear on click |
| `notes` | speaker notes |
| `small` · `muted` · `hot` · `mono` · `center` · `mark` | text styling |
| `scrim` | dark panel behind text over a photo |
| `no-wheel` | scroll wheel over this element doesn't change slides |
| `r-stack` · `r-hstack` · `r-vstack` · `r-stretch` · `r-fit-text` | reveal.js layout helpers |

---

## Limits and gotchas

- **Math** isn't set up yet (it needs a bundled KaTeX to work offline). Ask when you need it.
- **Step-through code highlighting** (reveal's `data-line-numbers="1|3-5"`) isn't available, because the kit uses pandoc's highlighter. For a walkthrough, put copies of the block in an `r-stack` and mark lines with `[…]{.mark}` in raw HTML, or show the code on consecutive slides.
- **Things that need the internet** (iframes, web fonts, CDN scripts, `data-background-video`) break the offline promise. Keep everything as files in the work folder.
- **IDs are global** across the deck: pandoc builds slide IDs from titles, so two slides titled "Demo" become `#demo` and `#demo-1`. Give widgets classes, not IDs.
- **Raw HTML blocks need a blank line** before and after, or pandoc may treat them as inline.
- **Images referenced from raw HTML** (`<img src="x.png">`) are embedded too, as long as the path is relative to the work folder.

---

## Customizing the kit

| To change… | Edit |
|---|---|
| Brand colors, fonts, sizes | the `:root` block at the top of `theme/lightturtles.css` |
| A layout's look | `theme/layouts/<name>.css` |
| A new layout | copy a layout file to `theme/layouts/mine.css`, then `layout: mine` |
| Code colors | `theme/code.theme` |
| Footer content | `js/chrome.html` |
| Slide size, transitions, numbering | `defaults/lightturtles.yaml` |
| New-deck template | `skeleton.md` |
| Logos | `theme/brand/*.svg` (convert EPS with `tools/eps2svg.sh`) |

Any rule you put in a deck's own `<style>` block overrides the theme for that deck only.
