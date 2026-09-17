# Module conventions

How we build HubSpot custom modules. Four documents and a starter folder, based
on what we learned shipping Blog cards and the Public consultation listing.

| File | Read it when |
| --- | --- |
| `CONVENTIONS.md` | Starting a module, or settling an argument about how to do something |
| `ACCESSIBILITY.md` | Building anything interactive, and before any accessibility review |
| `CHECKLIST.md` | Before every `hs upload` |
| `README-TEMPLATE.md` | Writing a module's own README |
| `_Starter.module/` | Copy it to start a new module |

## Starting a new module

```bash
cp -r _Starter.module ../YourModule.module
cd ../YourModule.module
```

Then:

1. Replace the `mod-` class prefix and the `data-mod*` attributes with your own.
   Three or four letters, unique in the theme. `mod-` is a placeholder and
   nothing should ship with it
2. Set `label` in `meta.json`, since that is what the editor sees
3. Replace the fields in `fields.json`. Keep the accessibility group
4. Build the markup, then the styles, then the script
5. Write the README from `README-TEMPLATE.md`
6. Work through `CHECKLIST.md`

The starter is a skeleton, not a feature. It carries the conventions, the token
block, the `:where()` reset, container queries, preference queries, the four JS
layers and the boot guard, with just enough markup to show where things go.

## The short version

If you read nothing else:

- Write the root class twice, keep custom properties off `:root`, reset with
  `:where()`
- Container queries, not media queries
- Nothing that changes after render belongs in HubL, because pages are cached
- Write attribute values in the format you plan to read, and parse carefully
- Every string an editor can see is a field, screen reader wording included
- Classes are for CSS, `data-` attributes are for JS
- One live region, and never disable the element that has focus
- Measure contrast instead of judging it by eye
