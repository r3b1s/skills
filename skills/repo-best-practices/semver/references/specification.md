# Semantic Versioning 2.0.0

Read when the rules, version format, precedence or a FAQ answer matter. This is a summary
of [the specification](https://semver.org/spec/v2.0.0.html), which is the authority.
Item numbers below are the specification's own.

## The rules

| Item | Rule |
|---|---|
| 1 | Software using SemVer must declare a public API, in code or documentation. It should be precise and comprehensive. |
| 2 | A normal version is `X.Y.Z`: non-negative integers with no leading zeroes. Each must increase numerically, so `1.9.0` is followed by `1.10.0`. |
| 3 | A released version's contents must not be modified. Any modification must be a new version. |
| 4 | `0.y.z` is initial development. Anything may change at any time, and the API should not be considered stable. |
| 5 | `1.0.0` defines the public API. How the number rises afterwards depends on that API and how it changes. |
| 6 | PATCH (x > 0) must rise if only backward-compatible bug fixes are introduced. A bug fix is an internal change that fixes incorrect behaviour. |
| 7 | MINOR (x > 0) must rise for new backward-compatible public functionality, and when anything public is marked deprecated. It may rise for substantial new private functionality or improvements. It may include patch-level changes. PATCH resets to 0. |
| 8 | MAJOR (x > 0) must rise for any backward-incompatible public change. It may include minor and patch changes. MINOR and PATCH reset to 0. |
| 9 | A pre-release may follow the patch: a hyphen, then dot-separated identifiers. It ranks below its normal version, and may not meet the compatibility its number implies. |
| 10 | Build metadata may follow: a plus sign, then dot-separated identifiers. It must be ignored for precedence. |
| 11 | Precedence is how versions are ordered. See [Precedence](#precedence). |

## Version format

```text
MAJOR.MINOR.PATCH[-PRERELEASE][+BUILD]
```

Identifiers in `PRERELEASE` and `BUILD` use only `[0-9A-Za-z-]` and must not be empty.
Numeric pre-release identifiers must not have leading zeroes. Build identifiers may.

| Version | Valid? | Why |
|---|---|---|
| `1.2.3` | yes | |
| `1.2.3-alpha.1` | yes | |
| `1.2.3-0a` | yes | contains a letter, so it is not a numeric identifier |
| `1.2.3+001` | yes | leading zeroes are allowed in build metadata |
| `1.2.3-rc.1+exp.sha.5114f85` | yes | pre-release, then build metadata |
| `1.2` | no | all three numbers are required |
| `v1.2.3` | no | a tag name; the semantic version is `1.2.3` |
| `01.2.3` | no | leading zero in a number |
| `1.2.3-01` | no | leading zero in a numeric pre-release identifier |
| `1.2.3-alpha..1` | no | empty identifier |
| `1.2.3-beta_1` | no | `_` is not allowed |

## Precedence

Compare in this order, and stop at the first difference:

1. MAJOR, MINOR and PATCH, numerically. Ignore build metadata throughout.
2. If those are equal, a version with a pre-release ranks below one without.
3. If both have pre-releases, compare their dot-separated identifiers left to right:
   - digits only: compare numerically
   - letters or hyphens present: compare as text, in ASCII order
   - a numeric identifier ranks below a non-numeric one
   - if all shared identifiers are equal, the version with more identifiers ranks higher

The specification's example order:

```text
1.0.0-alpha < 1.0.0-alpha.1 < 1.0.0-alpha.beta < 1.0.0-beta < 1.0.0-beta.2
  < 1.0.0-beta.11 < 1.0.0-rc.1 < 1.0.0
```

Two versions that differ only in build metadata have the same precedence.

## Validate a version

The specification suggests these patterns. They are reproduced unchanged.

With named groups, for PCRE, Python and Go:

```text
^(?P<major>0|[1-9]\d*)\.(?P<minor>0|[1-9]\d*)\.(?P<patch>0|[1-9]\d*)(?:-(?P<prerelease>(?:0|[1-9]\d*|\d*[a-zA-Z-][0-9a-zA-Z-]*)(?:\.(?:0|[1-9]\d*|\d*[a-zA-Z-][0-9a-zA-Z-]*))*))?(?:\+(?P<buildmetadata>[0-9a-zA-Z-]+(?:\.[0-9a-zA-Z-]+)*))?$
```

With numbered groups (1 major, 2 minor, 3 patch, 4 pre-release, 5 build metadata), for
JavaScript, PCRE, Python and Go:

```text
^(0|[1-9]\d*)\.(0|[1-9]\d*)\.(0|[1-9]\d*)(?:-((?:0|[1-9]\d*|\d*[a-zA-Z-][0-9a-zA-Z-]*)(?:\.(?:0|[1-9]\d*|\d*[a-zA-Z-][0-9a-zA-Z-]*))*))?(?:\+([0-9a-zA-Z-]+(?:\.[0-9a-zA-Z-]+)*))?$
```

Strip the `v` from a tag name before testing it. SemVer has no size limit on the string,
but specific systems may impose one.

## Sorting versions

General version sorts do not follow SemVer precedence. Checked with GNU `sort` and git
2.56.0 on 2026-11-01:

- `sort -V` and `git tag --sort=version:refname` both place `1.0.0` **before**
  `1.0.0-alpha`, which is the wrong way round. Taking the last line can therefore report a
  pre-release as the latest version.
- `git -c versionsort.suffix=- tag --sort=version:refname` places pre-releases before
  their normal version. It is still not a full SemVer comparison.

*Beyond the spec:* for anything that matters, use a SemVer library from the project's
ecosystem, or apply [Precedence](#precedence) directly.

## FAQ answers in brief

| Question | Answer |
|---|---|
| How do I handle 0.y.z? | Start at `0.1.0` and raise MINOR for each release. |
| When is 1.0.0? | If it is in production, has a stable API users depend on, or you worry a lot about backward compatibility, it should probably already be 1.0.0. |
| Does this slow development? | Major version zero exists for rapid development. If the API changes daily, stay on 0.y.z or use a separate branch for the next major version. |
| Won't I reach 42.0.0 quickly? | Incompatible changes should not be made lightly. Needing a major bump makes you weigh the cost for users. |
| I updated a dependency, API unchanged? | Compatible. PATCH if it fixes a bug, MINOR if it adds functionality. |
| I changed the API without matching the version? | Use judgment. With a large audience, a major release can be right even for a strict fix, because the number should convey what matters to users. |
| How do I deprecate? | Update the documentation, and issue a minor release with the deprecation. At least one such minor release should come before removal in a major release. |
| Is `v1.2.3` a semantic version? | No. It is a tag name, and `1.2.3` is the semantic version. |

## The accidental break answer

For an incompatible change released as a minor version, the published 2.0.0 page says to
"release a new minor version that corrects the problem and restores backward
compatibility". As read on 2026-11-01, that is the wording on both semver.org and its
`/spec/v2.0.0.html` page. The `semver.md` file on the `master` branch of
[semver/semver](https://github.com/semver/semver) says "patch" in the same sentence.

This skill follows the published 2.0.0 text. The FAQ is guidance, and the numbered items do
not settle this case. Both forms agree on what matters: restore compatibility in a new
version, never modify the bad release, and tell users about it.

## Attribution

Semantic Versioning was authored by Tom Preston-Werner and is licensed under
[CC BY 3.0](https://creativecommons.org/licenses/by/3.0/). This file summarises it in this
skill's own words, and reproduces its two regular expressions and short phrases unchanged.
