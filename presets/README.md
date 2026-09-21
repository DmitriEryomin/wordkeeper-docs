# Wordkeeper presets

The public `index.json` catalogue and its matching `<id>.json` files are served
from `/presets/` by GitHub Pages. Wordkeeper downloads them only when a person
opens Import presets or selects a list. Nothing in the local Dictionary is sent.

Each catalogue item has a stable lowercase `id`, localized `title` and
`description` (`en` and `de`), and an exact `wordCount`. The matching file has
the same `id`, a Category name, and an array of `{ "term", "translation" }`
pairs. `formatVersion` is currently 1. Keep existing IDs stable: changing a
published list changes what future imports receive. If a word pair already
exists in a local Dictionary, importing a revised list skips it.

The three initial lists are small, curated introductions rather than exhaustive
or officially certified CEFR vocabularies. The Armenian list uses Eastern
Armenian. Review translations and level placement with fluent speakers before
expanding the lists.
