# Module conventions

How we build HubSpot custom modules. Every rule here comes from something that
went wrong at least once.

Use this for any new module.

---

## Naming and layout

A module is a folder ending in `.module`. Name the folder after the module, in
PascalCase, with no spaces:

```
PublicConsultation.module/
  meta.json
  fields.json
  module.html
  module.css
  module.js
  README.md
```

Spaces work but then every CLI command needs quotes. Pick PascalCase and stick
to it.

HubSpot uploads everything in the folder. Keep test files and notes outside it.

**Class prefix.** Take three or four letters from the module name and use them
on every class and every `data-` attribute. `TeamGrid.module` becomes `tgd-`.
`EventCalendar.module` becomes `evc-`. Never use the same prefix twice. If two
modules share one, a style in the first starts changing the second.

The examples below use `mod-` as a placeholder. Swap in your own prefix. Nothing
should ship with `mod-` in it.

---

## CSS

**Write the root class twice in every rule.**

```css
.mod-outer.mod-outer .mod-card { … }
```

That gives 0,3,0 specificity, which beats almost any theme selector without
`!important`. Rules you add later need the same treatment, or the theme will win
on those.

**Put custom properties on the root element, not `:root`.**

```css
.mod-outer.mod-outer {
  --mod-card-radius: 18px;
  --mod-gap: 20px;
}
```

`:root` leaks onto the page and clashes with the next module. Field values reach
these through an inline `style` attribute on the root element, which also lets
two copies of the module on one page look different.

**Reset with `:where()`.**

```css
.mod-outer.mod-outer :where(div, ul, li, p, h2, h3, span, a, img, article,
aside, section, button, input, select, label, fieldset, legend) {
  margin: 0; padding: 0; border: 0; border-radius: 0;
  font-family: inherit; font-size: inherit; line-height: inherit;
  list-style: none; background: none; box-shadow: none;
  min-width: 0; max-width: none;
}
```

`:where()` has no specificity, so your own rules still win. Any property you
leave out is one the theme can set for you. The usual ones to remember are
`min-width`, `box-shadow` and `font-family`. List every tag you actually use. If
you forget `select`, your styled dropdown keeps a black native border.

**Use container queries, not media queries.** A module has no idea how wide its
column is. Put `container: name / inline-size` on the root, then write the
queries one level in. An element cannot answer a query about itself.

**Units.** Use `cqi` for anything that should grow with the module. Use `px`
when two elements need to wrap at the same width, because `ch` depends on each
element's own font size. A title at 20px and a paragraph at 15px both set to
`66ch` end up different widths, and the block looks crooked.

**Always add these three blocks:**

```css
@media (prefers-reduced-motion: reduce) { /* transitions to 0.01ms */ }
@media (prefers-contrast: more)          { /* real borders */ }
@media (forced-colors: active)           { /* real borders, since fills are dropped */ }
```

**`[hidden]` needs help.** If you set `display: flex` or `grid` on something, the
browser's own `[hidden]` rule stops working. Add
`.mod-thing[hidden] { display: none; }` next to it.

---

## HubL

**Load fonts in `require_head`** so the request happens once, however many copies
of the module are on the page:

```hubl
{% require_head %}
  <link rel="preconnect" href="https://fonts.gstatic.com" crossorigin>
  <link rel="stylesheet" href="https://fonts.googleapis.com/css2?family=…">
{% end_require_head %}
```

**Build every id from `name`.** `id="mod-{{ name }}"`,
`for="mod-search-{{ name }}"`. Fixed ids break as soon as someone adds the module
to a page twice.

**Keep anything that changes after render out of HubL.** A page is rendered once
and then cached, so a value worked out at render time is old by the time a
visitor sees it. That covers anything based on the clock, anything based on who
is looking, and anything counted. Work it out in `module.js`, which runs on every
visit. This is the most expensive mistake on the list.

**Write attribute values in the format you plan to read.** A field printed into
an attribute comes out however HubL felt like printing it, which is rarely what
`parseInt` or `new Date()` expects. Format it in the template, then parse it
carefully in the script.

**Escape anything going into an attribute:** `{{ value|escape }}`. Run rich text
through `|striptags` before putting it in a search index.

**Guard every optional field:** `{{ item.thing|default("", true) }}` before any
filter that expects text. A null reaching `|lower` or `|replace` kills the whole
expression, and one bad repeater row takes the rest of the loop with it.

**Check the inside of composite fields too:**
`{% if item.image and item.image.src %}`. Image and link fields come back null on
a new repeater row, so testing the outer object is not enough.

---

## fields.json

The validator's error messages are not helpful. These four rules cover most
failures:

- **`label` is a reserved field name.** So are `name`, `type`, `id` and
  `children`. Use `group_label`, `option_label`, `option_value` instead.
- **A repeated group cannot have a `default` array on the group itself.**
  HubSpot reads those objects as field definitions and reports
  "missing field name" or "'unknown' is not a valid field type". Put the
  defaults on the child fields and set `occurrence.default` for how many rows to
  start with.
- **`occurrence.default` has to be at least `occurrence.min`.** If `min` is 1 or
  more, set `default` yourself.
- **Some portals reject unusual keys inside repeaters,** such as `choices`,
  `responsive` and `supported_types`. If a group fails while plain field types
  pass, look there first.

**If an upload fails, bisect.** Cut the failing group down to one `text` field,
upload, then add fields back two at a time. The error path names the group.
Guessing takes longer.

**Make every string an editor can see a field.** Button text, empty states,
group headings, screen reader labels. A module with English in the template
cannot go on a Dutch page.

**Write help text in the editor's words,** including which values are allowed
and, on colour fields that sit behind text, the contrast to aim for.

---

## module.js

**Scope to the root element and guard against running twice.**

```js
document.querySelectorAll('[data-mod]').forEach(function (root) {
  if (root.dataset.modReady === 'true') return;
  root.dataset.modReady = 'true';
  new Listing(root).start();
});
```

Also listen for `hsPageEditorUpdate`, so the editor preview keeps working:

```js
window.addEventListener('message', function (e) {
  if (e.data && e.data.type === 'hsPageEditorUpdate') boot();
});
```

**Split it into layers.** Even a small module gets these four. Each one knows
nothing about the layer above it.

| Layer | Holds |
| --- | --- |
| Value logic | Parsing and plain calculation. No DOM |
| Domain objects | One per repeated thing. The only code that touches that thing's DOM |
| Views | One region of the page each. No business logic. They report up and get told what to show |
| Controller | The only place that knows about all the parts |

The benefit is real: a new filter, sort or view mode becomes a new class plus one
line in the controller, and you edit nothing that already works.

**Pass in anything you cannot control,** such as the clock or randomness, as a
constructor argument with a sensible default. Logic that reaches for a global can
only be tested in whatever state that global is in.

**Parse carefully at the edges.** Anything read from a `data-` attribute is text
of unknown shape. Write one parser, use it everywhere, and return null rather
than something wrong.

**No libraries.** querySelector, classList, dataset and addEventListener cover
all of this. A module that drags jQuery into a theme that dropped it becomes a
support ticket.

**Let CSS handle animation and JS handle state.** Toggle a class. Never set
`hidden` on something you are trying to animate, because the animation stops
straight away.

**Do not rewrite the DOM when nothing changed.** Re-appending an element takes it
out of the page and puts it back, so anything focused inside it loses focus.
Check whether the order actually changed before reordering.

---

## The contract between the files

The `data-` attributes are the interface between `module.html` and `module.js`.
Write them in the README and keep them steady. That is what lets you change where
the data comes from, from a repeater to HubDB, without touching the script.

```
data-mod              root element
data-mod-ready        set by JS, stops it running twice
data-mod-item         one repeated thing
data-mod-<region>     a region the script owns (list, results, panel)
data-<value>          a value the script reads off an item
```

Classes are for CSS. `data-` attributes are for JS. A script that hooks
`.mod-card` breaks the day someone renames a class for visual reasons.

---

## Performance

- Run images through `resize_image_url` at two or three widths, so a 4000px
  upload is not sent to a 300px card. This only works for images in the HubSpot
  file manager
- Add `loading="lazy"` and `width` and `height` to every image, so the space is
  reserved
- No network requests beyond fonts. No analytics, no CDN scripts
- Cards render on the server, so the content is there even if the script fails.
  That also means a broken script can look like a CSS problem

---

## Before you upload

Use `CHECKLIST.md`. It is short, and it catches the things this document is too
long to re-read.
