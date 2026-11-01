# Calendar Versioning

Read when you need a term, a notation rule or an FAQ answer. This is a factual summary of
[calver.org](https://calver.org/), spec 26.0 (page dated 2026-09-11), which is the authority.

## Version segments

| Segment | Meaning |
|---|---|
| Major | the first number, and the most common place for a calendar value |
| Minor | the second number |
| Micro | the third and usually last number, also called the patch segment |
| Modifier | an optional text tag, such as `dev`, `alpha`, `beta` or `rc1` |

Most versions have two or three numeric segments plus the optional modifier. Four numeric
segments are discouraged by convention.

## Date tokens

| Token | Meaning | Examples |
|---|---|---|
| `YYYY` | full year | `2026` |
| `YY` | short year, relative to 2000 | `26`, or `6` for 2006 |
| `0Y` | zero-padded year, relative to 2000 | `26`, or `06` for 2006 |
| `M` | short month | `3`, `10` |
| `0M` | zero-padded month | `03`, `10` |
| `W` | short week, from the start of the year | `7`, `41` |
| `0W` | zero-padded week | `07`, `41` |
| `D` | short day | `9`, `24` |
| `0D` | zero-padded day | `09`, `24` |

`MINOR` and `MICRO` in a scheme are ordinary counters.

## Notation rules

- A single letter is the plain number. A leading `0` marks zero-padding.
- `YY` and `0Y` differ only for the years 2000 to 2009, and from 2100.
- Date segments start at 1. Traditional counters start at 0.
- A trailing optional segment goes in square brackets: `YYYY.0M.0D[.MICRO]`.
- The Gregorian calendar and UTC are assumed. Another calendar is possible if the project
  states it.

### Deprecated spellings

From 2016 until spec 26.0, the page wrote the short month, week and day as `MM`, `WW` and
`DD`. Those spellings keep that old meaning, so in existing badges and tool configuration
`YY.MM.MICRO` means `YY.M.MICRO`.

They are deprecated because most date formats read a doubled letter as two digits. Write `M`,
`W` and `D`, or `0M`, `0W` and `0D`. Tools that implement the doubled spellings still work,
including bumpver, bump-my-version and JReleaser. Prefer the single letters as tools add them.

### Weeks

Weeks are mutually exclusive with months and days. Definitions differ, and a year can have
up to 54 numbered weeks depending on the definition, so a project using them must state
which it means.

| `strftime` | Definition |
|---|---|
| `%W` | Monday-started weeks, from 00 |
| `%U` | Sunday-started weeks, from 00 |
| `%V` | ISO 8601 weeks, 01 to 53, with their own week-based year |

*Beyond the spec:* `%V` pairs with the ISO year `%G`, not `%Y`, because the first and last
days of a year can belong to a neighbouring ISO year.

## FAQ answers in brief

| Question | Answer |
|---|---|
| When to use CalVer? | When third parties use the project, use a serious version. CalVer suits a large or constantly changing scope, and time-sensitive projects driven by outside events. |
| What about breaking changes? | CalVer deliberately does not communicate them, since there has never been agreement on what counts as one, and some versioned things have none. Use a changelog and a policy, and pair the version with a time-based deprecation policy. |
| Several releases a day? | Append a rising micro segment: `YYYY.M.D.MICRO`. Intra-day timestamps cost readability and can still collide. |
| To pad or not? | Unpadded is the sensible default, unless versions commonly appear as filenames. Padded segments are invalid SemVer. Pad when plain text sorting needs to work, so `26.04` stays ahead of `26.10`. |

## Case studies

| Project | Scheme | Why it fits |
|---|---|---|
| Ubuntu | `YY.0M` | many parts, and support periods can be worked out from the version |
| Apple operating systems | `YY.MINOR`, year offset by one | one number across platforms, shows how current a device is |
| NVIDIA GPU Operator | unpadded, for example `22.9.0` | a new major every six months and twelve months of support; unpadded to stay valid SemVer |
| NVIDIA RAPIDS, NGC, Legate | padded, for example `21.06.00` | versions are container tags that sort as text |
| Twisted | `YY.M.MICRO` | a large project whose parts deprecate separately, with time-based removal |
| yt-dlp | `YYYY.0M.0D[.MICRO]` | fast, broad scope; PyPI lists the normalized `2026.8.19` for tag `2026.08.19` |
| IANA time zone database | `YYYY` plus a letter | changes follow political decisions, not a schedule |
| certifi | `YYYY.0M.0D` | certificate updates have no fixed schedule |
| Stripe API | `YYYY-0M-0D.MODIFIER` | integrations pin a dated API version |

## Badges

Any project can make a [shields.io](https://shields.io/) static badge, with the scheme in the
middle of the URL, linked to calver.org so readers can decode it:

```markdown
[![CalVer - YYYY.M.D](https://img.shields.io/badge/calver-YYYY.M.D-22bfda.svg)](https://calver.org/)
```

In that URL, dots need no escaping, a literal dash is written twice (`--`), and square
brackets are percent-encoded (`%5B`, `%5D`).

## Spec history

| Revision | Change |
|---|---|
| 26.0 (2026) | `M`, `W`, `D` replace the doubled spellings; square brackets for optional segments; padding guidance; refreshed case studies |
| 19.7 (2019) | revised text, case studies and users |
| 16.6 (2016) | original publication |

## Attribution

Calendar Versioning is published at [calver.org](https://calver.org/), maintained at
[mahmoud/calver](https://github.com/mahmoud/calver), and licensed on the site under
[CC BY-SA 4.0](https://creativecommons.org/licenses/by-sa/4.0/). This file does not adapt
that text. It records the notation and facts in this skill's own words, so read the original
for the full discussion.
