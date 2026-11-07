# Stories

GitHub Pages serves `/stories/index.json` and `/stories/<id>.json`. The app fetches
only when browsing or choosing a story. Dictionary data is never uploaded.
Publish these files before releasing a client with the Stories feature.

The catalogue uses `format: "wordkeeper-stories"`, `formatVersion: 1`, and a
`stories` array. Each listing has a stable lowercase hyphenated `id`, supported
`languageCode` and `translationLanguageCode`, English/German `title` and
`description` (each at most 160 characters), CEFR `level`, and integer
`readingMinutes` from 1 to 60. The catalogue supports up to 200 stories.

Each story uses `format: "wordkeeper-story"`, `formatVersion: 1`, the exact listing
metadata, and a source-language `name` (at most 50 characters, also used as its
Dictionary Category). Include 1–200 vocabulary `words` with unique stable `id`,
`term`, and `translation`. Terms and translations follow the Dictionary's
200-character limit. Duplicate normalized term/translation pairs are rejected.

`paragraphs` contains 1–100 arrays of segments. A segment is `{ "text": "..." }`
or `{ "text": "...", "wordId": "vocabulary-id" }`. Preserve spaces and
punctuation in the text. Only referenced segments are interactive. Every
vocabulary entry must appear in the story; every reference must resolve.
References can map inflections or phrases to a canonical saved term. No HTML,
Markdown, executable content, or external translation service is used.

Keep each payload under 250,000 characters and paragraph data under 150,000
characters. Paragraphs support at most 500 segments, each at most 10,000
characters. Read the launch files for complete examples.

The two launch stories are original Spanish fiction with Russian translations,
approximately 300 words each. Have fluent speakers review language, translations,
and CEFR placement before release. Additional languages may be added using codes
supported by the app. Translation language is fixed per story, independent of the
interface language.

Users retain one validated offline story and its paragraph position. Published
updates affect future downloads; already downloaded stories remain unchanged.
Replacing a story never removes saved Dictionary Words or Practice Progress.
Story state is excluded from Dictionary backups. Existing `/presets/` formats
and catalogues are independent and unchanged.

## Validate before publishing

From the app repository, with dependencies installed, run:

```sh
STORY_CONTENT_DIR=../wordkeeper-docs/stories npx jest --runInBand __tests__/stories-test.ts
```

This validates the catalogue, every launch payload, vocabulary links, metadata,
and the 250–400-word launch length. The launch-content test is skipped during
ordinary app tests when `STORY_CONTENT_DIR` is absent, so an app checkout does not
require a sibling docs checkout.
