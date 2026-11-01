# Keep a Changelog 1.1.0

Read when writing or converting a changelog. This is a summary of
[Keep a Changelog 1.1.0](https://keepachangelog.com/en/1.1.0/), which is the authority.
It describes itself as a carefully considered opinion, not a standard.

A changelog is a file holding a curated, chronologically ordered list of notable changes
for each version of a project. Its purpose is to let users and contributors see precisely
what notable changes were made between releases.

## File name

Call it `CHANGELOG.md`. Some projects use `HISTORY`, `NEWS` or `RELEASES`, but a consistent
name makes it easier for users to find.

## Guiding principles

- Changelogs are for humans, not machines.
- There is an entry for every single version.
- The same types of change are grouped.
- Versions and sections are linkable.
- The latest version comes first.
- Each version shows its release date.
- It states whether the project follows Semantic Versioning.

*Beyond the spec:* a project on another scheme, such as Calendar Versioning, names that
scheme in the header instead. The version headings then use that scheme's numbers.

## Layout

```markdown
# Changelog

All notable changes to this project will be documented in this file.

The format is based on [Keep a Changelog](https://keepachangelog.com/en/1.1.0/),
and this project adheres to [Semantic Versioning](https://semver.org/spec/v2.0.0.html).

## [Unreleased]

### Fixed

- Stop the importer dropping the last row of a file with no trailing newline.

## [1.2.0] - 2026-10-14

### Added

- `--format json` on the `export` command.

### Deprecated

- `--json`. Use `--format json` instead; `--json` will be removed in 2.0.0.

## [1.1.1] - 2026-09-30

### Security

- Reject archive paths containing `..`, which could write outside the target directory.

[Unreleased]: https://github.com/OWNER/REPO/compare/v1.2.0...HEAD
[1.2.0]: https://github.com/OWNER/REPO/compare/v1.1.1...v1.2.0
[1.1.1]: https://github.com/OWNER/REPO/releases/tag/v1.1.1
```

- Version headings use the version without a `v`. Links use the tag name.
- Dates are `YYYY-MM-DD`. This format runs from largest unit to smallest, does not overlap
  ambiguously with regional formats, and follows an ISO standard.
- Each version links to a comparison with the previous one, and the oldest links to its
  own tag. These URLs are GitHub's. Use your forge's equivalent.
- Leave out types with no entries. The Keep a Changelog project removed empty sections from
  its own changelog because they add noise, and missing ones are assumed to have no notable
  changes.

## Types of change

| Type | For |
|---|---|
| Added | new features |
| Changed | changes in existing functionality |
| Deprecated | features soon to be removed |
| Removed | features now removed |
| Fixed | bug fixes |
| Security | vulnerabilities |

*Beyond the spec:* use these six and do not add others, so the file stays predictable. In
the sections, order entries by how much they matter to users.

## Unreleased

Keep an `Unreleased` section at the top. People can see what to expect in the next release,
and at release time its entries move into a new version section.

## Writing entries

| Instead of | Write |
|---|---|
| `fix: handle nil in parser (#482)` | Fix a crash when parsing an empty configuration file. |
| `refactor auth module` | Leave it out if users cannot observe it. |
| `update deps` | Leave it out, unless users are affected. If so, say how. |
| `change default timeout` | **Breaking:** The default request timeout is now 30 seconds, down from 120. Set `timeout` to keep the old value. |

*Beyond the spec:* the `**Breaking:**` marker, and the rule that each entry states the
effect on users.

## Bad practices

- **Commit log diffs.** Commits record steps in the code's evolution, and are full of noise
  such as merge commits and obscure titles. An entry records a noteworthy difference, often
  across several commits, for end users.
- **Ignoring deprecations.** An upgrade should make it painfully clear what will break. If
  nothing else, list deprecations, removals and breaking changes.
- **Confusing dates.** Use the date format above.
- **Inconsistent changes.** Mentioning only some changes can be as dangerous as having no
  changelog, because users may take it as the single source of truth. Not every change
  needs recording (a single whitespace removal may not), but every important one does.

*Beyond the spec:* tools that draft a changelog from commits are fine as input, as long as
a person or agent curates the result against these rules. With
[Conventional Commits](https://www.conventionalcommits.org/en/v1.0.0/), `feat` usually
suggests Added or Changed, and `fix` suggests Fixed or Security.

## Yanked releases

Yanked releases are versions pulled because of a serious bug or security issue. They should
still appear in the changelog, in this form:

```markdown
## [0.0.5] - 2014-12-13 [YANKED]
```

The tag is loud so people notice it, and its brackets make it easy to parse.

## Rewriting a changelog

This is allowed. Add missing releases to an unmaintained changelog, and add a breaking
change you forgot to mention. It does not break the SemVer rule against modifying a release:
*Beyond the spec:* correct the file on the default branch, and leave the released tag and
its contents alone.

## Forge release pages

Keep a Changelog notes that GitHub Releases can turn tags into rich release notes, but that
they create a non-portable changelog that displays only within GitHub, and are less
discoverable than a top-level file. *Beyond the spec:* use both, with `CHANGELOG.md` as the
source and the release page holding a copy of the version's section.

## Automatic parsing

The source calls this difficult, because projects use very different formats and file
names. *Beyond the spec:* following this format closely is what makes a project's own file
parseable, for example to extract one version's section at release time.

## Check a changelog

*Beyond the spec:* this pattern matches a released version heading, with groups 1 version,
2 date and 3 the yanked marker. `Unreleased` is the one heading that has no date.

```text
^## \[([^\]]+)\] - (\d{4}-\d{2}-\d{2})( \[YANKED\])?$
```

## Attribution

Keep a Changelog is created and maintained by Olivier Lacan, designed by Tyler Fortune, and
MIT licensed. This file summarises it in this skill's own words. The header text in the
layout above, and the `[YANKED]` example, are the source's own.
