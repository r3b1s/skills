---
name: calver
description: Version software with Calendar Versioning (CalVer), using the calver.org 26.0 notation. Use when deciding whether date-based versions fit a project, choosing or documenting a scheme such as YYYY.MINOR.MICRO, YY.0M or YYYY.M.D, moving a project from SemVer to CalVer, computing the next version, deciding on zero-padding, handling several releases in one day, setting a time-based deprecation policy, or reading CalVer notation, badges and tool configuration.
license: MIT
metadata:
  spec-version: "26.0"
  verified-on: "calver.org, spec 26.0 dated 2026-09-11, read 2026-11-01"
---

# Version software with CalVer

## Scope

This skill helps a person, an agent, or both version a project by its release calendar.

CalVer is a shared vocabulary, not one scheme. It names the parts of a date-based version,
and each project designs the scheme that fits it.

- **Source:** [Calendar Versioning](https://calver.org/), spec 26.0. Where this skill and
  the source disagree, the source wins.
- **Marking:** anything the source does not settle is marked *Beyond the spec*. That is this
  skill's guidance, and a project may decide otherwise.
- **Applies to** projects whose releases follow a calendar, or whose scope makes a
  compatibility promise impractical. Package ecosystems limit the schemes you can use; see
  [references/ecosystems.md](references/ecosystems.md).
- **Not covered:** Semantic Versioning, and how to write changelogs. If a project already
  uses another scheme, keep it unless the human decides to change.

## Ground rules

1. **The human decides; you propose.** Stop for approval before adopting CalVer, choosing a
   scheme, setting a deprecation or support policy, and anything outward-facing or hard to
   reverse, such as tagging, pushing or publishing.
2. **Write the current notation.** `M`, `W` and `D` are unpadded; `0M`, `0W` and `0D` are
   zero-padded. Never write `MM`, `WW` or `DD`. Reading them is different: they are the
   deprecated spellings of the **unpadded** forms, so `YY.MM.MICRO` means `YY.M.MICRO`, not
   a two-digit month.
3. **Promise nothing through the number.** CalVer deliberately does not communicate breaking
   changes. A changelog and a policy do that.
4. **Never change a released version.** *Beyond the spec:* any change is a new version. Do
   not move or reuse a published tag, and do not retag old commits with calendar numbers.
5. **Report the reasoning.** Give the proposed version, the date and time zone it came from,
   and what you did and did not verify.

## Choose the task

| Task | Go to |
|---|---|
| Decide whether CalVer fits | [Decide](#decide) |
| Design a scheme | [Choose a scheme](#choose-a-scheme) |
| Write the scheme down | [Document it](#document-it) |
| Work out the next version | [Compute the next version](#compute-the-next-version) |
| Change from another scheme | [Move from another scheme](#move-from-another-scheme) |
| Retire functionality | [Handle breaking changes](#handle-breaking-changes) |
| Read notation or look up a term | [references/specification.md](references/specification.md) |

## Decide

CalVer's FAQ says it is a strong choice if either applies:

- **Large or constantly changing scope:** big systems and frameworks, loose sets of
  utilities, or audiences who should not have to judge whether a change is breaking.
- **Time-sensitive releases driven by outside events:** business needs such as support
  schedules, security updates, or political changes.

*Beyond the spec:* prefer Semantic Versioning for a library whose users rely on a
compatibility signal, especially where tools resolve updates from the version number. See
[references/ecosystems.md](references/ecosystems.md).

## Choose a scheme

1. **Start from the release rhythm.** The schemes and projects are from calver.org.
   *Beyond the spec:* matching them to a rhythm is this skill's guidance, and some example
   numbers are illustrative.

   | Rhythm | Scheme | Example | Used by |
   |---|---|---|---|
   | A few releases a year | `YYYY.MINOR.MICRO` | `2026.2.0` | JetBrains IDEs |
   | A year signal plus counters | `YY.MINOR.MICRO` | `26.2.1` | pip, CockroachDB |
   | Roughly monthly | `YYYY.M.MICRO` | `2026.9.0` | Home Assistant |
   | Roughly monthly, shorter | `YY.M.MICRO` | `26.4.0` | Twisted |
   | Scheduled platform releases | `YY.0M` | `26.04` | Ubuntu |
   | Released whenever events demand | `YYYY.0M.0D[.MICRO]` | `2026.08.19` | yt-dlp |

2. **Keep it short.** Two or three numeric segments, plus an optional modifier. Four numeric
   segments are discouraged, though an optional trailing counter is a known pattern, written
   in square brackets.
3. **Avoid weeks unless they fit.** Weeks exclude months and days, and have several
   definitions. If you use them, state which one.
4. **Decide on padding.** Unpadded is the sensible default: `YYYY.M.D`, not `YYYY.0M.0D`.
   Pad only when versions commonly appear in filenames, image tags or object-store keys that
   get sorted as text. A padded segment is invalid SemVer, which some ecosystems enforce.
5. **Check the ecosystem.** See [references/ecosystems.md](references/ecosystems.md) before
   proposing anything for a published package. For example, npm rejects `2026.4`, and needs
   three numbers.

## Document it

State these in the README, near the top of a Versioning section:

- that the project uses CalVer, with a link to calver.org
- the scheme, in current notation, with one real example naming each segment
- the calendar and time zone, if not the assumed Gregorian and UTC
- the week definition, if weeks are used
- the deprecation or support policy, since the number does not carry one

```markdown
## Versioning

This project uses [Calendar Versioning](https://calver.org/). Versions follow
`YYYY.M.MICRO`: `2026.9.2` is the third release of September 2026, in UTC. Deprecated
features are removed no sooner than one year and two releases after the deprecation.
```

A badge is optional; see [references/specification.md](references/specification.md#badges).

## Compute the next version

1. **Take the release date in UTC,** the assumed convention. Near midnight, the UTC date can
   differ from your local one.
   ```sh
   date -u +%Y.%-m.%-d   # YYYY.M.D
   date -u +%y.%m        # YY.0M
   ```
2. **Fill the date segments,** which start at 1. Traditional counters start at 0.
   *Beyond the spec:* some projects count `MINOR` from 1, as in `2026.1`, so follow what the
   project does.
3. **Raise the counter for another release in the same period.** *Beyond the spec:* reset
   `MICRO` to 0 when the date part changes: `2026.9.2` is followed by `2026.10.0`.
4. **For several releases in a day,** append a rising `MICRO`: `2026.9.11`, then
   `2026.9.11.1`. Avoid timestamps inside the day, which read badly and can still collide.
   This fourth segment is invalid in SemVer-based ecosystems, so use `YYYY.M.MICRO` there.
5. **Verify.** *Beyond the spec:* the version must sort above every earlier release in the
   tool that consumes it, not just by eye, and it must match in every place it is recorded.

## Move from another scheme

1. **Check that the numbers still rise.** *Beyond the spec:* the first calendar version must
   sort above the current one. A long-lived project on `30.2.0` cannot move to `26.4.0`,
   because that goes backwards, but it can use `YYYY`. Python also offers version epochs for
   this; see [references/ecosystems.md](references/ecosystems.md).
2. **Start from the next release.** NVIDIA's GPU Operator went from `1.11` to `22.9.0`, and
   RAPIDS from `0.19` to `21.06.00`. Old versions keep their numbers.
3. **Update every place the version is recorded,** and add a changelog note naming the new
   scheme.
4. **Warn users of update tools.** *Beyond the spec:* tools built for Semantic Versioning read
   a new first number as a breaking change, so users on `^` ranges will not update
   automatically. See [references/ecosystems.md](references/ecosystems.md).

## Handle breaking changes

CalVer does not signal them, because there has never been agreement on what counts as one.
Its answer is that nothing replaces a changelog and a policy.

1. **Set a time-based deprecation policy.** Twisted's is the source's model: one year and two
   releases between deprecating something and removing it.
2. **Announce deprecations** in the changelog and documentation, with the earliest removal
   date.
3. **Remove only after the window,** and say so plainly in the changelog.
4. **Consider a support window.** Ubuntu and NVIDIA's GPU Operator use theirs so that a
   version's date tells users when support ends.

With CalVer, a commit marked as breaking does not raise any number. *Beyond the spec:* keep
marking it anyway, so it reaches the changelog.

## Pre-releases

The source allows an optional text modifier such as `dev`, `alpha`, `beta` or `rc1`, but does
not fix its format. *Beyond the spec:* use the form the ecosystem expects: `2026.9.0rc1` in
Python, and `2026.9.0-rc.1` in SemVer-based tools.

## Reference

- [references/specification.md](references/specification.md): segment and token definitions,
  notation rules, weeks, FAQ answers, case studies, badges, and the spec's history.
- [references/ecosystems.md](references/ecosystems.md): what npm, Cargo, Go, Python, Linux
  packaging and container tags allow, and how each sorts versions.
