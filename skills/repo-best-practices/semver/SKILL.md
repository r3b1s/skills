---
name: semver
description: Version software with Semantic Versioning 2.0.0 (SemVer). Use when adopting SemVer, declaring a public API, deciding whether a change is breaking, choosing the next MAJOR.MINOR.PATCH version, validating or comparing version strings, planning deprecations or pre-releases, tagging a release, or recovering from a wrongly numbered release.
license: MIT
metadata:
  spec-version: "2.0.0"
  verified-on: "semver.org/spec/v2.0.0.html, read 2026-11-01"
---

# Version software with SemVer

## Scope

This skill helps a person, an agent, or both give a project version numbers that tell users
what an upgrade will do to them.

- **Source:** [Semantic Versioning 2.0.0](https://semver.org/spec/v2.0.0.html). Where this
  skill and the source disagree, the source wins.
- **Marking:** anything the source does not settle is marked *Beyond the spec*. That is
  this skill's guidance, and a project may decide otherwise.
- **Applies to** anything with consumers who rely on compatibility: libraries, CLIs,
  services, plugins, file formats. Package ecosystems add their own rules; see
  [references/ecosystems.md](references/ecosystems.md).
- **Not covered:** other schemes such as Calendar Versioning, and how to write changelogs
  or commit messages. If a project already uses another scheme, keep it unless the human
  decides to change.

## Ground rules

1. **The human decides; you propose.** Stop for approval before: adopting SemVer, declaring
   what the public API is, releasing 1.0.0 or any major version, and anything outward-facing
   or hard to reverse, such as tagging, pushing or publishing.
2. **Read the changes, not only the commit messages.** A message can miss a breaking change.
   Base the version on the diff against the public API.
3. **Never change a released version.** Its contents stay as they are, and any change is a
   new version (item 3). Do not move, reuse or delete a published tag.
4. **Report the reasoning.** Give the proposed version, the change that decided it, and what
   you did and did not verify.

## Choose the task

| Task | Go to |
|---|---|
| Set up SemVer in a project | [Adopt](#adopt) |
| Work out the next version number | [Choose the next version](#choose-the-next-version) |
| Publish a version | [Tag a release](#tag-a-release) |
| Retire part of the public API | [Deprecate, then remove](#deprecate-then-remove) |
| Publish a test version | [Pre-releases](#pre-releases) |
| Recover from a bad release | [Repair](#repair) |
| Check or compare version strings | [references/specification.md](references/specification.md#validate-a-version) |

## Adopt

1. **See what exists.** List tags (`git tag --list`), version fields in manifests, and the
   versions already published to any registry.
2. **Declare the public API.** SemVer cannot work without one (item 1), and it may live in
   code or documentation. Propose what is covered and what is not, using
   [references/public-api.md](references/public-api.md).
3. **Choose the starting version.**
   - New project: `0.1.0`, the simplest start according to the FAQ.
   - Already used in production, or users depend on a stable API: it should probably already
     be `1.0.0`, by the same FAQ.
   - Existing versions: *Beyond the spec:* continue from them. The next version must have
     higher precedence than every version already published. Leave the old tags alone, even
     if they are not valid SemVer.
4. **Say so in the README.** State that the project follows SemVer, link to it (the SemVer
   site asks for this), and state the public API or link to where it is declared.
5. **Keep one source for the version.** *Beyond the spec:* keep the number in as few places
   as the ecosystem allows, and make a release check that the manifest and the tag agree.

## Choose the next version

Compare against the last release, not the last commit. The highest result wins, and it
resets the lower numbers to zero.

1. **Any backward-incompatible change to the public API:** MAJOR.
2. **New backward-compatible functionality, or anything in the public API newly marked as
   deprecated:** MINOR. Marking a deprecation is required to raise MINOR (item 7).
3. **Only backward-compatible bug fixes:** PATCH.
4. **Nothing users can observe:** *Beyond the spec:* no release is needed. If one is made
   anyway, use PATCH.

*Beyond the spec:* other records can suggest a bump. Treat them as evidence, because the
public API decides. Work that never shipped does not count, such as a fix to an unreleased
feature, or a breaking change reverted before the release.

| Signal | Usual effect from 1.0.0 onward |
|---|---|
| Changelog `Added`, or a `feat` commit | MINOR |
| Changelog `Changed` | MINOR if compatible, MAJOR if not |
| Changelog `Deprecated` | MINOR |
| Changelog `Removed` | MAJOR if it was public, otherwise none |
| Changelog `Fixed`, or a `fix` commit | PATCH |
| Changelog `Security` | whatever the fix is: usually PATCH, MAJOR if it must break compatibility |
| A commit marked `!` or `BREAKING CHANGE` | MAJOR |

Cases the FAQ settles:

- **Updating your own dependency, API unchanged:** compatible. PATCH if it was to fix a bug,
  MINOR if it brings new functionality.
- **A fix breaks people who relied on the wrong behaviour:** use judgment. With a large
  affected audience a major release can be right, even if the change is strictly a fix.

**Below 1.0.0.** Items 6 to 8 apply only from 1.0.0 onward. In `0.y.z`, anything may change
at any time (item 4), and the FAQ's simplest approach is to raise MINOR on each release.
*Beyond the spec:* raise `y` for breaking changes and `z` for compatible fixes. That stays
within SemVer and matches how npm and Cargo resolve `0.y` ranges, so users of those tools do
not receive breaking changes automatically. Never let a breaking change turn `0.y.z` into
`1.0.0` by itself.

**When to reach 1.0.0.** Propose it once the software is in production, users depend on a
stable API, or you are worrying a lot about backward compatibility. Those are the FAQ's
signs. The human decides.

For full rules, precedence and validation, see
[references/specification.md](references/specification.md). For hard calls on what counts as
breaking, see [references/public-api.md](references/public-api.md).

## Tag a release

1. **Find the base.** `git describe --tags --abbrev=0` gives the last tag reachable from
   here. Check it against the published versions, and see
   [how versions sort](references/specification.md#sorting-versions) before sorting tags.
2. **Review everything since then.** Read `git log <tag>..HEAD` and `git diff <tag>..HEAD`.
3. **Choose the version** as above.
4. **Update the version** in the manifests, any lock files and any changelog.
5. **Commit, then stop for approval.** *Beyond the spec:* commit before tagging, so the tag
   contains the version change.
6. **Tag and publish, once approved.** The tag is `vX.Y.Z` and the version is `X.Y.Z`: the
   `v` is part of the tag name, not of the semantic version.
   ```sh
   git tag -a vX.Y.Z -m "Release version X.Y.Z"
   git push origin vX.Y.Z
   ```
7. **Verify.** The version matches the
   [SemVer pattern](references/specification.md#validate-a-version), has higher precedence
   than every earlier release, and is the same in the manifest and the tag.

## Deprecate, then remove

1. Update the documentation so users know about the change, and name the replacement.
2. Release a new MINOR version with the deprecation in place.
3. Remove it only in a later MAJOR version.

The FAQ asks for at least one minor release containing the deprecation before the removal,
so users can move smoothly.

## Pre-releases

A pre-release is the target version plus a hyphen and dot-separated identifiers, such as
`2.0.0-rc.1`. It ranks below `2.0.0`, and it signals that the version may not meet the
compatibility its number implies.

- *Beyond the spec:* use `alpha`, `beta`, then `rc`, each with a dot and a counter:
  `2.0.0-alpha.1`, `2.0.0-beta.1`, `2.0.0-rc.1`. They sort correctly under SemVer.
- **Keep the dot.** Dot-separated numbers compare numerically, so `rc.11` ranks above
  `rc.2`. Without the dot the identifier compares as text, so `rc11` ranks below `rc2`.
- Check [references/ecosystems.md](references/ecosystems.md) first. Python accepts only
  alpha, beta and rc forms, and normalizes them.

## Repair

A released version never changes, so every repair is a new release, a note, or both.

| Situation | What to do |
|---|---|
| An incompatible change shipped as a minor or patch | Fix it and release a new **minor** version that restores compatibility, as the published 2.0.0 FAQ says. Document the bad version and tell users. See [the note on this answer](references/specification.md#the-accidental-break-answer). |
| The incompatible change is now relied on by many users | Use judgment. A new major version that keeps the new behaviour may do less harm. The human decides. |
| A release had a serious bug or security issue | Publish a fixed version, and warn users about the bad one. Use the registry's own yank or deprecate feature if it has one. |
| The wrong number was published or tagged | Do not retag. Publish a correct new version, and note what happened. |

## Reference

- [references/specification.md](references/specification.md): the rules in full, version
  format, precedence, validation patterns, sorting, and the FAQ answers.
- [references/public-api.md](references/public-api.md): what counts as public for each kind
  of project, and how to judge whether a change is breaking.
- [references/ecosystems.md](references/ecosystems.md): where npm, Cargo, Python packaging
  and Go modules differ from plain SemVer.
