# I18n Extractor

A Camillia skill that finds hard-coded UI strings in React and .NET code, moves them into resource files with stable hierarchical keys, wires the lookups, and handles plurals and interpolation correctly.

## What the agent gains

- Concrete search patterns for JSX text, JSX attributes, Razor views, and C# literals
- A false-positive skip list so CSS classes and log messages stay untouched
- A stable key scheme (`area.component.element`) that survives copy edits
- Before/after examples for react-i18next, react-intl (ICU), and `IStringLocalizer`, including plurals

## Install

Install from the Camillia Community Hub. After installation the skill lives in the workspace under `.workspace/extensions/i18n-extractor/` and the agent applies it whenever you localize a component or a review flags hard-coded strings.

## Contents

- `extensions/i18n-extractor/SKILL.md` - the procedure the agent follows
- `extensions/i18n-extractor/extraction-examples.md` - skip list and before/after pairs per framework

## Validate

```
node scripts/validate.mjs
```

## License

MIT - see [LICENSE](LICENSE). Copyright 2026 Camillia contributors.
