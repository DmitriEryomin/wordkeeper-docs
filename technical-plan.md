# Wordkeeper product and technical plan

## Product goal

Wordkeeper replaces a paper notebook for manually recording foreign-language words and their translations. It is intentionally private and local-first: there is no account, backend, automatic synchronization, analytics, or app-owned cloud storage.

The mobile MVP supports iOS and Android and allows a user to:

- Add, edit, and delete a word and its translation.
- Store up to 1,000 words on the free plan.
- Create, rename, and delete categories.
- Move a word between a category and Uncategorized.
- Search and filter the dictionary.
- Export a portable backup and import it later.
- Use an interface that follows the device or per-app language.

Literal handwriting recognition, translation APIs, learning exercises, accounts, synchronization, and web support are outside the MVP.

## User journeys

### Add and organize a word

1. Open the Dictionary screen.
2. Open Add word and enter a foreign word and its translation.
3. Save the entry locally. New entries start in Uncategorized.
4. Create a category such as Food.
5. Edit or move the word into Food.

### Export a backup

1. Open the Dictionary menu and choose Export dictionary.
2. Wordkeeper creates a versioned JSON file in temporary storage.
3. The native share sheet lets the user save or share the file.

The app never uploads a backup automatically. The user controls its destination.

### Import a backup

1. Choose Import dictionary and select one JSON backup with the native document picker.
2. Wordkeeper validates the file without changing the database.
3. A preview shows words, categories, duplicates, and the expected result.
4. The user merges the backup or explicitly replaces the current dictionary.
5. The import is applied atomically or rolled back completely.

## Stack

- Expo SDK 57, React Native 0.86, React 19.2, and strict TypeScript.
- Expo Router with a stable native Stack.
- `expo-sqlite` for persistent relational storage.
- React Context and hooks for the small in-memory view of database state.
- `expo-localization` and `i18n-js` for interface localization.
- `expo-file-system`, `expo-document-picker`, and `expo-sharing` for backups.
- React Native primitives and `FlatList`; no design system or external state library.
- Expo Go for development before any custom or store build.

## Navigation

```text
Dictionary
├── Add word                 modal
├── Edit word                modal
├── Categories
│   ├── Create category      modal
│   └── Rename category      modal
└── Import backup            modal preview
```

Routes live under `src/app`; components, data access, domain rules, backup logic, and translations live outside the route directory.

## Data model

### categories

| Column | Type | Rule |
| --- | --- | --- |
| id | INTEGER | Primary key |
| name | TEXT | Required, original spelling |
| name_key | TEXT | Unicode-normalized, unique |
| created_at | INTEGER | Unix milliseconds |
| updated_at | INTEGER | Unix milliseconds |

### words

| Column | Type | Rule |
| --- | --- | --- |
| id | INTEGER | Primary key |
| term | TEXT | Required, original spelling |
| translation | TEXT | Required, original spelling |
| term_key | TEXT | Unicode-normalized comparison value |
| translation_key | TEXT | Unicode-normalized comparison value |
| category_id | INTEGER or NULL | Foreign key to categories |
| created_at | INTEGER | Unix milliseconds |
| updated_at | INTEGER | Unix milliseconds |

Uncategorized is represented by a null `category_id`, not a special category row. Deleting a category sets its words back to Uncategorized. An exact normalized word-and-translation pair is unique, while the same word may have a different translation.

The free plan's 1,000-word allowance is an entitlement rule rather than a physical storage limit. New word and import flows enforce it, while SQLite can retain an existing larger dictionary after a Premium subscription expires. Reading, editing, deleting, and exporting existing words never depend on the entitlement.

## Architecture

```text
Route screens
    ↓
Feature components and hooks
    ↓
Dictionary provider
    ↓
Domain validation and normalization
    ↓
Word and category repositories
    ↓
expo-sqlite
```

- Screens own presentation and navigation.
- The dictionary provider exposes state and user actions.
- Domain functions own normalization, validation, duplicate behavior, and entitlement limits.
- Repositories own all SQL and return domain-shaped data.
- SQLite is the durable source of truth.
- The root `SQLiteProvider` opens the database and runs versioned migrations before rendering the application.

Migrations enable foreign keys and WAL mode and use SQLite `user_version`. User values are always bound parameters; bulk SQL execution is reserved for static migration statements.

## Localization

The initial interface languages are English (`en`) and German (`de`). The app walks the device's preferred locale list and selects the first supported language, falling back to English. It declares supported locales through the `expo-localization` config plugin so iOS and Android can expose native per-app language selection.

On Android the locale is refreshed whenever the app returns to the foreground. iOS restarts the app when its language changes. All user-facing strings, navigation titles, dialogs, errors, accessibility labels, and pluralized counters use translation keys. User-entered words, translations, and category names are never translated.

Layouts use logical start/end behavior so future right-to-left translations do not require a structural rewrite.

## Backup format and rules

Backups are readable JSON rather than raw SQLite files:

The internal format identifier remains `wdict-backup` so that backups made before the rename remain compatible.

```json
{
  "format": "wdict-backup",
  "formatVersion": 1,
  "exportedAt": "2026-08-19T12:00:00.000Z",
  "appVersion": "1.0.0",
  "categories": [{ "id": 1, "name": "Food" }],
  "words": [
    {
      "term": "apple",
      "translation": "яблоко",
      "categoryId": 1,
      "createdAt": 1787137200000,
      "updatedAt": 1787137200000
    }
  ]
}
```

The import validator rejects invalid JSON, unsupported versions, incorrect types, and empty or oversized values. Free and expired plans also reject backups with more than 1,000 words; Premium accepts larger dictionaries.

Merge is the default safe action. It merges categories by normalized name and skips exact duplicate pairs. If merging would exceed 1,000 unique words, it is blocked rather than silently dropping entries. Replace requires explicit destructive confirmation. Both modes run in a single database transaction.

## Acceptance criteria

- A saved word survives app restarts and works in airplane mode.
- Free users cannot add or import beyond 1,000 words; an existing larger dictionary remains accessible after Premium expires.
- Armenian, Russian, German, accented, and composed/decomposed Unicode text round-trip unchanged.
- Category deletion moves its words to Uncategorized.
- Search matches both terms and translations.
- English and German have the same translation keys.
- English devices show English; German devices show German; unsupported languages fall back to English.
- Exporting and importing into an empty database reproduces all words and categories.
- Merge skips duplicates and preserves existing data.
- A malformed or failed import leaves the current dictionary unchanged.
- Replace is never performed without confirmation.
- Essential flows are accessible with VoiceOver/TalkBack and dynamic text sizing.

## Delivery order

1. Localization, database schema, migrations, repositories, and domain rules.
2. Dictionary list, add/edit/delete word flows, search, filters, and word counter.
3. Category creation, rename, deletion, and word assignment.
4. Versioned export and import preview with merge/replace.
5. Focused automated tests, Expo lint/type-check, and iOS/Android Expo Go verification.
