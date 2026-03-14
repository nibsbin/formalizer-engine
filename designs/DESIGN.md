# Typst Formalizer Design Doc

A publishable Typst package that renders pixel-perfect PDF form replicas from a PyMuPDF-extracted schema.

## Layers

| Layer | Responsibility |
|---|---|
| **Orchestration** (separate project) | PyMuPDF → `FIELDS.json`, PDF → page PNGs, codegen of `copied-form.typ` |
| **This package** | Rendering engine: schema + values dict → overlaid form |
| **Generated `copied-form.typ`** | Named-parameter function wrapping the package |
| **End user** | Calls `#form(first_name: "John", agree_terms: true)` |


## Package API

```typst
#let render-form(schema, backgrounds, values: (:)) = { ... }
```

- `schema` — result of `json("FIELDS.json")`
- `backgrounds` — list of PNG paths, one per page
- `values` — dict of field name → value (omit to render blank)

## Rendering

1. Group fields by page
2. Per page: fixed-size `block` → `image(bg, fit: "stretch")` → `place(dx, dy)` per field
3. Coordinates: PyMuPDF bbox is top-left origin, same as Typst — multiply by `1pt`
4. Field widgets are transparent overlays (no chrome); PNG provides visual styling

## Generated `copied-form.typ`

Orchestration codegens this once from `FIELDS.json`. End users edit it to fill the form.

```typst
#import "@preview/formalizer:0.1.0": render-form

#let form(
  first_name:  "",      // text
  agree_terms: false,   // checkbox
  choice:      none,    // radio
  gender:      "Male",  // combobox [Male, Female]
  items:       "Aaa",   // listbox [Aaa, Bbb]
) = render-form(
  schema: json("FIELDS.json"),
  backgrounds: ("page1.png",),
  values: (first_name: first_name, agree_terms: agree_terms, choice: choice, gender: gender, items: items),
)
```

End user:
```typst
#import "copied-form.typ": form
#form(first_name: "John", agree_terms: true)
```

## Testing

Use the `uv` project manager to bootstrap a python pytest environment in tests/. Write mixed .py and .typ tests with your discretion to robustly validate typst-formalizer package functionality.