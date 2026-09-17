# Accessibility baseline

We build to WCAG 2.2 Level AA. These are the patterns that come up again and
again, written as things to copy rather than rules to interpret.

`mod-` in the examples stands in for the module's own class prefix.

---

## Structure

**Give the region a name.** A `<section>` with no name does not show up in a
screen reader's list of landmarks. Add a hidden heading and point at it:

```html
<section aria-labelledby="mod-heading-{{ name }}">
  <h2 class="mod-sr-only" id="mod-heading-{{ name }}">{{ module.region_name }}</h2>
```

Make the name a field. Only the editor knows what this block is called on their
page.

**Use real headings.** Card titles are `<h3>` under that `<h2>`, not styled
divs. If the module might sit under different page structures, make the level a
field.

**Wrap groups of checkboxes in `fieldset` and `legend`.** A heading above a list
of inputs is not a group as far as a screen reader is concerned.

**Put list semantics back when you remove bullets.** Safari drops them once you
set `list-style: none`, so add `role="list"` to any `<ul>` you have styled flat.

---

## Values that need a label

A single word in a card, like "Closed" or "Project", means nothing on its own.
Add a hidden label:

```html
<span class="mod-tag"><span class="mod-sr-only">Type: </span>{{ item.tag }}</span>
```

If the script rewrites that value, write into an inner span so the label
survives:

```html
<span class="mod-status" data-badge>
  <span class="mod-sr-only">Status: </span><span data-badge-text></span>
</span>
```

When a symbol carries meaning, say it in words as well. A screen reader may skip
a dash, a slash or an arrow completely:

```html
<span aria-hidden="true">–</span><span class="mod-sr-only"> to </span>
```

Use the element that exists for the job, such as `<time>`, `<address>` or
`<abbr>`, rather than a styled span.

---

## Counts and other small numbers

A number inside a `<label>` becomes part of the control's name, so the checkbox
reads as "Project 3". Hide the visible number and put a spoken version in a
description instead:

```html
<label class="mod-check">
  <input type="checkbox" aria-describedby="opt-project-count">
  <span class="mod-check-text">Project</span>
  <span class="mod-facet-count" aria-hidden="true">3</span>
</label>
<span class="mod-sr-only" id="opt-project-count">3 matching</span>
```

The unit matters. "3" on its own could mean anything. Make that word a field so
it can be translated.

---

## Never disable the element that has focus

Disabling the focused element drops focus to `<body>`, and the person loses
their place halfway through a task. When state makes a control unavailable:

```js
input.setAttribute('aria-disabled', dead ? 'true' : 'false');
input.disabled = dead && document.activeElement !== input;
```

The same goes for controls you remove. If something focused is about to go, move
focus somewhere sensible first, not to the top of the page.

---

## Panels that open and close

```html
<button aria-expanded="false" aria-controls="panel-{{ name }}">…</button>
<aside id="panel-{{ name }}" aria-hidden="true" inert>…</aside>
```

- `aria-expanded` on the button, `aria-controls` pointing at the panel
- A closed panel is `inert`, and switches to `visibility: hidden` once its
  animation finishes. A panel that is clipped but still visible stays in the tab
  order
- Escape closes it and puts focus back on the button
- Set `inert` as a property where the browser supports it, and fall back to the
  attribute where it does not

---

## Status messages

**One live region per module.** Two of them talk over each other and the person
hears neither. Pick the one that sums up the change, usually a results count:

```html
<p aria-live="polite" aria-atomic="true">3 of 5 consultations</p>
```

`aria-atomic` makes it read the whole sentence instead of only the digits that
changed. Everything else stays quiet.

---

## Links and buttons

- The name has to say where the link goes: "Read more: <title>", with the
  visible text first so voice control users can say what they see
- Links that open a new tab say so in hidden text
- Buttons with only an icon need hidden text, and the icon gets `aria-hidden`

---

## Focus

Every control gets a visible ring, `<select>` included. That means adding
`select` to both the reset and the focus rule:

```css
.mod-outer.mod-outer :is(a, button, input, select, [tabindex]):focus-visible {
  outline: 3px solid var(--mod-focus-color);
  outline-offset: 3px;
}
```

Custom checkboxes hide the real input, so put the ring on the visible box:

```css
.mod-check input:focus-visible + .mod-box { outline: 3px solid …; }
```

Where a ring sits on a photo, add a white band under it so it stays visible.

---

## Colour contrast

Measure it. Do not judge by eye. You need 4.5:1 for normal text and 3:1 for
large text and the edges of controls. Small white text on a coloured badge is
where modules usually fail, because mid greens, oranges and reds all look fine
and all fall short.

```python
def lum(h):
    h = h.lstrip('#'); c = [int(h[i:i+2],16)/255 for i in (0,2,4)]
    c = [x/12.92 if x <= 0.03928 else ((x+0.055)/1.055)**2.4 for x in c]
    return 0.2126*c[0] + 0.7152*c[1] + 0.0722*c[2]

def ratio(a, b):
    la, lb = lum(a), lum(b)
    return (max(la,lb) + 0.05) / (min(la,lb) + 0.05)
```

Run every pair of text and background in the module, including light text on the
accent colour and muted text on cards. Write the failures and their replacements
in the README, so the next person can see the values were chosen on purpose.

Editors can still pick colours that fail. Put the target ratio in the help text
of any colour field that sits behind text.

---

## Preference queries

```css
@media (prefers-reduced-motion: reduce) { /* transitions to 0.01ms */ }
@media (prefers-contrast: more)          { /* real borders */ }
@media (forced-colors: active)           { /* real borders */ }
```

In forced colours mode the browser drops backgrounds and shadows, so anything
whose shape came from a fill needs a real border to survive.

---

## What no tool can check

- **Alt text.** The editor writes it, and a page full of `hero-final-2.jpg`
  passes every automated test while failing everyone who needs it. Say so in the
  README and in the field's help text
- **Tabbing through the module once** with a screen reader on, in the page it
  actually ships on
- **Whether the reading order makes sense.** A card can be labelled perfectly
  and still read in the wrong order
