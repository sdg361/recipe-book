# Recipe Book

A single-file, offline-capable HTML recipe book. Published via GitHub Pages at
<https://sdg361.github.io/recipe-book/> from `main` / root.

## Files

- `Recipe Book.html` — the whole app: CSS, one JSON data block, and the renderer. No build step.
- `index.html` — tiny redirect to `Recipe%20Book.html` (GitHub Pages entry point).

## Where the data lives

All content is in a single `<script type="application/json" id="BOOK">…</script>` block
(currently line 152). Everything after it is the renderer; the JSON is the only part to edit
for content changes.

```
{
  "menu":    { "title": str, "sections": [ … ] },
  "recipes": { "<slug-id>": { … } },
  "updated": "YYYY-MM-DD"
}
```

### Menu sections

A section has `name` plus either `items` (flat list) or `groups`
(`[{ "name": str, "items": [ … ] }]`, used by "Other Western Mains").

Item (a dish on the menu):

| key | req | meaning |
|---|---|---|
| `t` | yes | dish text as shown, e.g. `"Tuna melt w/ tuna salad, cheese"` |
| `c` | yes | cuisine label; drives the filter chips, so reuse an existing string exactly |
| `r` | no | list of recipe ids linking to `recipes`; a dish with none renders as plain text |
| `sub` | no | list of variant lines shown beneath the dish |
| `x` | no | `true` = flagged for weekday cooking; renders a small "X" badge (no legend in the page) |

### Recipes

Keyed by a kebab-case id. `title`, `src`, `serves`, `unit`, `groups`, `steps` are present on
every recipe; the rest are optional.

| key | meaning |
|---|---|
| `title` | recipe heading |
| `sub` | one-line description |
| `src` | original source file name, e.g. `"Tuna melt.docx"` |
| `serves` | number — the base for the servings scaler |
| `unit` | what `serves` counts: `"servings"`, `"sandwich"`, … |
| `servesNote` | free text shown instead of the bare number, e.g. `"2–3"` |
| `scaleNote` | caveat about scaling |
| `groups` | ingredient groups: `[{ "n": "Tuna salad", "i": [ … ] }]` |
| `steps` | list of instruction strings |
| `meta` | `[{ "k": "Prep", "v": "5 min" }, …]` |
| `notes` | list of note strings |
| `lang` | source language of the text, e.g. `"de"` |
| `transcribed` | `true` when transcribed from a photo/scan |

Ingredient (`groups[].i[]`):

- `n` — name, required. On its own it renders as an unquantified line ("Salt and pepper, to taste").
- `q` — quantity: a number, or `[min, max]` for a range. Scales with servings.
- `u` — unit (`g`, `tbsp`, `slices`, `can`, …).
- `note` — parenthetical, e.g. `"optional"`.

## Editing

Edit the JSON with a script rather than by hand — it is one long line. Re-serialise with
`json.dumps(d, ensure_ascii=False, separators=(',', ':'))`, which reproduces the existing
formatting byte-for-byte, so diffs stay limited to real changes:

```python
import re, json
p = 'Recipe Book.html'
h = open(p, encoding='utf-8').read()
m = re.search(r'(id="BOOK">)(.*?)(</script>)', h, re.S)
d = json.loads(m.group(2))
# … mutate d …
out = json.dumps(d, ensure_ascii=False, separators=(',', ':'))
open(p, 'w', encoding='utf-8').write(h[:m.start(2)] + out + h[m.end(2):])
```

After editing:

1. `python3 -I -c "import re,json; json.loads(re.search(r'id=\"BOOK\">(.*?)</script>', open('Recipe Book.html',encoding='utf-8').read(), re.S).group(1))"` to confirm the JSON still parses.
2. Check every id in a dish's `r` exists in `recipes` (and that new recipe ids are unique).
3. Bump `updated` to today's date — the footer renders it as "Last updated …".
4. Open the file in a browser and confirm the header count and the new entry render.
   Local `file://` URLs can't be driven by the in-app browser; serve the folder instead
   (`python3 -m http.server`) or check the live site after pushing.
5. Commit and push to `main` — Pages redeploys automatically.

The header count (dishes · with recipes · recipe pages) and the cuisine chips are derived at
runtime, so nothing needs updating by hand.
