---
name: i18n-extractor
description: Use when asked to extract hard-coded UI strings from React or .NET code into resource files with stable hierarchical keys, wiring lookups, plurals, and interpolation.
---

# I18n Extractor

Move user-facing strings out of code into resource files without changing behavior.

## When to use / When not to use

Use this skill when:

- The user asks to localize a component or page, or a review flags hard-coded strings.
- A feature is being prepared for a new locale.

Do not use this skill when:

- The strings are developer-facing only (log messages, exception details, telemetry). Those stay in code.
- The repo has no i18n infrastructure at all. Set that up first, or ask the user which framework to adopt, before extracting.

## Inputs

- The target area: files or components the user named, or the files changed in the current diff.
- The i18n stack, detected from the repo:
  - React: `react-i18next` (`useTranslation`, JSON resources) or `react-intl` (`FormattedMessage`, ICU messages).
  - .NET: `IStringLocalizer` with `.resx` files, or a JSON-based localization setup.
- The existing resource files and their key hierarchy.

## Procedure

1. Find candidates in the target files:

   ```
   rg -n ">[A-Z][A-Za-z ,.!?]{2,}<" src/ --glob "*.tsx"
   rg -n '(placeholder|title|aria-label|alt)="[A-Za-z]' src/ --glob "*.tsx"
   rg -n '"[A-Z][^"]{2,}"' --glob "*.cshtml" --glob "*.razor"
   ```

   Also scan `.cs` files for user-facing literals: validation messages, notifications, email subjects.
2. Filter false positives before touching anything: CSS classes, `data-testid`, route paths, log and telemetry strings, and already-extracted `t(...)` / `_localizer[...]` calls stay in code. The full skip list is in [extraction-examples.md](extraction-examples.md).
3. Choose keys using the repo's hierarchy, or the scheme `<area>.<component>.<element>` with lower camel-case segments, for example `orders.detail.submitButton`. Keys must be stable: never derive them from the English text, or a wording tweak breaks translations.
4. Add the strings to the base resource file only (usually `en`): `src/i18n/en.json` for react-i18next, the matching `.resx` for .NET. Leave other locales to translators; do not machine-translate unless asked.
5. Wire the lookups:
   - React: `const { t } = useTranslation();` then `<button>{t("orders.detail.submitButton")}</button>`.
   - .NET: inject `IStringLocalizer<T>` and use `_localizer["orders.detail.submitButton"]`; in Razor views use `@inject IViewLocalizer Localizer`.
   Full before/after pairs are in [extraction-examples.md](extraction-examples.md).
6. Handle interpolation and plurals:
   - Interpolation: react-i18next uses `{{name}}` placeholders, for example `t("cart.greeting", { name })`; .NET uses `{0}` positional placeholders through the localizer.
   - Plurals: react-i18next suffixes `key_one` / `key_other` (plus `_zero` where a locale needs it); react-intl uses ICU `{count, plural, one {# item} other {# items}}`. Never build plurals with `count === 1 ? "item" : "items"` in code.
7. Re-scan the target files to confirm no candidate remains, then build and run the tests to catch missed keys or broken markup. Missing keys surface as raw key names at runtime; grep the console output for them.
8. Report: strings extracted, keys added, files touched, and anything deliberately left in code with the reason.

## Output format

- Edited source files with lookups wired, plus the updated base resource file.
- A summary table `file | key | string`, and a "left in code" list with a one-line reason per skipped candidate.

## Guardrails

- Behavior-preserving only: no copy edits, no rewording, no markup changes beyond the lookup wiring.
- One locale (the base) per change; never overwrite existing translations in other locale files.
- Do not invent a new key for a string that already exists in the resource file; reuse the existing key.
- Keep placeholder syntax exactly as the framework expects (`{{name}}` vs `{0}`); mixing them breaks interpolation silently.
- Workspace rules and the repository's existing key hierarchy and resource layout take precedence over the scheme above.
