# Checklist before uploading

Run through this before `hs upload`. It is short on purpose. The reasoning is in
`CONVENTIONS.md` and `ACCESSIBILITY.md`.

## Structure

- [ ] Folder is `PascalCase.module` and holds only the five module files plus
      `README.md`
- [ ] Class prefix is unique to this module and used on every class and
      `data-` attribute
- [ ] `meta.json` has a readable `label`, since that is what the editor sees

## CSS

- [ ] Root class written twice in every rule
- [ ] Custom properties on the root element, nothing on `:root`
- [ ] `:where()` reset lists every tag the markup uses, including `select`
- [ ] Breakpoints are container queries
- [ ] `[hidden]` overridden wherever `display` is set
- [ ] Blocks for reduced motion, more contrast and forced colours are there

## HubL

- [ ] Fonts loaded in `require_head`
- [ ] Every id built from `{{ name }}`
- [ ] Nothing that changes after render is worked out in HubL
- [ ] Attribute values written in the format the script reads
- [ ] Attribute values escaped, rich text stripped before indexing
- [ ] Every optional field guarded, including the inside of image and link fields

## fields.json

- [ ] No field called `label`, `name`, `type` or `id`
- [ ] No `default` array on a repeated group itself
- [ ] `occurrence.default` is at least `occurrence.min` everywhere
- [ ] Every string an editor can see is a field, screen reader wording included
- [ ] Help text says which values are allowed and what contrast to aim for

## JS

- [ ] Scoped to the root element, with a ready flag
- [ ] `hsPageEditorUpdate` listener in place
- [ ] Layers kept apart: values, domain, views, controller
- [ ] No libraries, no network calls
- [ ] Class toggles drive animation, not `hidden`
- [ ] Nothing rewrites the DOM when the order has not changed

## Accessibility

- [ ] The region has a name
- [ ] Heading levels are real and in order
- [ ] Checkbox groups use `fieldset` and `legend`
- [ ] Single values carry a hidden label, such as "Status: "
- [ ] Semantic elements used where they exist, and symbols that carry meaning
      have spoken text
- [ ] Small numbers are descriptions, not part of a control's name
- [ ] No control is disabled while it has focus
- [ ] Panels: `aria-expanded`, `aria-controls`, `inert` when closed, Escape puts
      focus back
- [ ] Exactly one live region, marked `aria-atomic`
- [ ] Link names say where they go, visible text first
- [ ] Focus ring on everything, `select` included
- [ ] Every text and background pair measured, failures fixed and written down

## By hand

- [ ] Tab through the whole module. No traps, focus always visible
- [ ] Listen to one card with a screen reader. Does the order make sense?
- [ ] Resize from 320px to 1600px and watch the container breakpoints
- [ ] Block JS and check the content still renders
- [ ] Put two copies on one page and check they do not interfere
- [ ] Open it in the HubSpot editor preview, not only locally

## README

- [ ] Files table, install command, settings by tab
- [ ] "How it works" covers the decisions that look wrong without context
- [ ] The `data-` attribute contract is written down
- [ ] Accessibility section with the contrast table
- [ ] Browser support and content limits
