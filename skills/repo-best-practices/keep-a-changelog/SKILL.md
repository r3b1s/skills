---
name: keep-a-changelog
description: Keep a human-readable CHANGELOG.md following Keep a Changelog 1.1.0. Use when creating or converting a changelog, writing entries, sorting changes into Added, Changed, Deprecated, Removed, Fixed and Security, maintaining an Unreleased section, moving entries into a version at release time, marking a release as yanked, correcting an existing changelog, or writing release notes from one.
license: MIT
metadata:
  spec-version: "1.1.0"
  verified-on: "keepachangelog.com/en/1.1.0/, read 2026-11-01"
---

# Keep a changelog

## Scope

This skill helps a person, an agent, or both keep a changelog that tells people what changed
between releases and what they need to do about it.

- **Source:** [Keep a Changelog 1.1.0](https://keepachangelog.com/en/1.1.0/). Where this
  skill and the source disagree, the source wins.
- **Marking:** anything the source does not settle is marked *Beyond the spec*. That is this
  skill's guidance, and a project may decide otherwise.
- **Works with any version scheme.** The format is written around Semantic Versioning, but
  the changelog itself does not depend on it. The number comes from the project's scheme.
- **Not covered:** choosing version numbers and writing commit messages.

## Ground rules

1. **The human decides; you propose.** Stop for approval before replacing an existing
   changelog, and before anything outward-facing, such as publishing a release page.
2. **Do not invent entries.** Every entry must trace to a real change. If a change's effect
   on users is unclear, say so and ask.
3. **Curate; do not paste.** The changelog is for people. Commits are raw material, never
   the entry text.
4. **Report what you checked.** State which changes you reviewed, and what you left out and
   why.

## Choose the task

| Task | Go to |
|---|---|
| Start or convert a changelog | [Create](#create) |
| Add to the changelog | [Record a change](#record-a-change) |
| Turn `Unreleased` into a version | [Release a version](#release-a-version) |
| Mark a pulled version | [Yank a release](#yank-a-release) |
| Fix or fill in an existing changelog | [Correct a changelog](#correct-a-changelog) |

## Create

1. **See what exists.** Look for `CHANGELOG`, `HISTORY`, `NEWS` or `RELEASES`, and list the
   project's tags and published versions.
2. **Start from the template.** Copy [assets/CHANGELOG.md.template](assets/CHANGELOG.md.template)
   to `CHANGELOG.md` in the repository root. Use that name, so users can find it.
3. **State the version scheme.** The format asks you to mention whether you follow Semantic
   Versioning. *Beyond the spec:* if the project uses another scheme, such as Calendar
   Versioning, name that scheme in the header instead.
4. **Handle existing history.** *Beyond the spec:* backfill past versions only from what tags
   and diffs show, and keep older history as it is if it cannot be reconstructed honestly.
5. **Link it** from the README.

## Record a change

Add notable changes to `## [Unreleased]` as you make them, so the release does not depend on
reconstruction later. People can also see what to expect next.

1. Put each entry under one of the six types, and omit types with no entries:

   | Type | For |
   |---|---|
   | Added | new features |
   | Changed | changes in existing functionality |
   | Deprecated | features soon to be removed |
   | Removed | features now removed |
   | Fixed | bug fixes |
   | Security | vulnerabilities |

2. Write for the people who use the project: what changed for them, and what they need to
   do. Do not paste commit messages.
3. *Beyond the spec:* mark breaking changes with a leading `**Breaking:**`, and say how to
   migrate. The format does not define a marker, but it calls deprecations, removals and
   breaking changes the minimum a changelog must list.
4. Leave out what users cannot observe, such as refactors, test changes and CI changes.

For examples, see
[references/specification.md](references/specification.md#writing-entries).

## Release a version

1. **Review everything since the last release.** Read `git log <tag>..HEAD` and
   `git diff <tag>..HEAD`, then reconcile them with `Unreleased`: add what is missing, and
   remove what is not notable. An entry for something that never shipped does not belong,
   such as a fix to a feature that is also unreleased.
2. **Get the version number** from the project's versioning scheme.
3. **Move the entries.** Under an emptied `## [Unreleased]`, add `## [X.Y.Z] - YYYY-MM-DD`
   with the release date. Use the version without a `v`, and put the latest version first.
4. **Update the link definitions** at the foot of the file: point `[Unreleased]` at a
   comparison from the new tag to `HEAD`, and add one for the new version comparing it with
   the previous tag.
5. **Copy the section to any release page.** *Beyond the spec:* keep `CHANGELOG.md` as the
   source. To extract one version's section:
   ```sh
   awk '/^## \[1\.2\.0\]/{f=1;next} /^(## |\[[^]]+\]: )/{f=0} f' CHANGELOG.md
   ```
6. **Verify.** Every version has an entry and a date, the latest comes first, and every link
   resolves. The [heading pattern](references/specification.md#check-a-changelog) can check
   the first two.

## Yank a release

A yanked release is one pulled because of a serious bug or security issue. Keep it in the
changelog, and mark it loudly:

```markdown
## [0.0.5] - 2014-12-13 [YANKED]
```

*Beyond the spec:* add the version that replaces it, and say why it was pulled.

## Correct a changelog

A changelog may be improved at any time. Add missing releases, and add any breaking change
that was left out. *Beyond the spec:* correct the file on the default branch, and leave
released tags and their contents alone.

## Reference

- [references/specification.md](references/specification.md): the format, principles, entry
  examples, antipatterns, yanked releases, and forge release pages.
- [assets/CHANGELOG.md.template](assets/CHANGELOG.md.template): a starting `CHANGELOG.md`.
