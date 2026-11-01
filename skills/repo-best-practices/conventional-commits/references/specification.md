# Conventional Commits 1.0.0

Read when writing, checking or reading commit messages in a project that uses this format.
This is a summary of [the specification](https://www.conventionalcommits.org/en/v1.0.0/),
which is the authority. Where the specification meets SemVer or a changelog, it leaves gaps,
and [Where the specifications meet](#where-the-specifications-meet) fills them.

Conventional Commits is a convention for commit messages. It gives tools an explicit history
to work from and dovetails with SemVer by naming features, fixes and breaking changes.

## Format

```text
<type>[optional scope][optional !]: <description>

[optional body]

[optional footer(s)]
```

## The rules

| Part | Rule |
|---|---|
| Type | Required. A noun such as `feat` or `fix`, followed by the optional scope, the optional `!`, then a colon and a space. |
| `feat` | Must be used when a commit adds a new feature to the application or library. |
| `fix` | Must be used when a commit represents a bug fix for the application. |
| Other types | Allowed, for example `docs`. They are not mandated, and have no implicit effect on the version unless they include a breaking change. |
| Scope | Optional. A noun for a section of the codebase, in parentheses: `fix(parser):`. |
| Description | Required, immediately after the colon and space. A short summary of the code changes. |
| Body | Optional. It starts one blank line after the description, and is free-form, with any number of paragraphs. |
| Footers | Optional, one blank line after the body. Each is a word token, then `: ` or ` #`, then a value. A value may contain spaces and newlines, and ends at the next valid token and separator. |
| Footer tokens | Use `-` in place of whitespace, as in `Acked-by`. `BREAKING CHANGE` is the one exception. |
| Breaking change | Must be marked by `!` immediately before the colon, by a footer, or both. It can be part of a commit of any type. |
| Breaking footer | Uppercase `BREAKING CHANGE`, then a colon, a space and a description. `BREAKING-CHANGE` is a synonym. |
| With `!` alone | The footer may be omitted, and the commit description then describes the breaking change. |
| Case | Implementors must not treat these units as case-sensitive, except `BREAKING CHANGE`, which must be uppercase. |

The specification mentions `@commitlint/config-conventional`, based on the Angular
convention, as recommending `build`, `chore`, `ci`, `docs`, `style`, `refactor`, `perf` and
`test`. These are examples. A team may define its own types and change them over time.

## Examples

```text
feat(export): add JSON output
```

```text
fix: stop the importer dropping the last row

Files with no trailing newline lost their final record, because the
reader flushed only on a newline.

Refs: #482
Reviewed-by: Sam
```

```text
feat(cli)!: require an explicit output path

The output path used to default to the working directory, which
overwrote files without warning.

BREAKING CHANGE: `export` now fails without `--output`. Pass
`--output .` to keep the old behaviour.
```

```text
revert: remove the retry queue

Refs: 676104e, a215868
```

| Message | Valid? | Why |
|---|---|---|
| `feat: add search` | yes | |
| `fix(parser): handle empty input` | yes | |
| `refactor!: rename the public client` | yes | any type can be breaking |
| `feat(api)!: drop v1 routes` | yes | `!` comes after the scope |
| `Feat: add search` | yes | case does not matter here, though consistency helps |
| `feat:add search` | no | the colon needs a following space |
| `feat!(api): drop v1 routes` | no | `!` must be immediately before the colon |
| `feat (api): drop v1 routes` | no | the scope follows the type directly |
| `feat(): add search` | no | an empty scope is not a noun |
| `add search` | no | no type |

Footer traps:

- `breaking change: new config` is not a breaking footer, because it must be uppercase.
- `BREAKING CHANGE #482` is not one either. That form needs a colon and a space.
- *Beyond the spec:* a final paragraph that starts with a word and a colon, such as
  `Note: this is slow`, matches the footer shape. Reword it if the commit body must stay a
  body.

## Validate a message

The specification gives no pattern. *Beyond the spec:* these are this skill's own, and they
are stricter on one point: they take a type to be letters only.

The first line, with groups 1 type, 2 scope, 3 `!`, 4 description:

```text
^([A-Za-z]+)(?:\(([^()\r\n]+)\))?(!)?: (\S.*)$
```

A breaking footer line:

```text
^BREAKING[ -]CHANGE: \S
```

To find marked breaking changes since a tag:

```sh
git log --format='%h %s' <tag>..HEAD | grep -E '^[0-9a-f]+ [A-Za-z]+(\([^)]*\))?!: '
git log --format='%h %s' -E --grep='^BREAKING[ -]CHANGE: ' <tag>..HEAD
```

These find what commits declare. They cannot find a breaking change nobody marked.

## Where the specifications meet

The table under [Version bumps](#version-bumps) comes from the specification. The rest of
this section is *Beyond the spec*.

### Version bumps

| Commit | Version effect |
|---|---|
| `fix` | PATCH |
| `feat` | MINOR |
| `!` or `BREAKING CHANGE`, on any type | MAJOR |
| Any other type | none by itself |

Across all commits since the last release, the highest effect wins. These cases need care:

- **Below 1.0.0.** The specification says to proceed as if already released, so keep marking
  breaking changes. Do not map them to MAJOR, though: that would turn `0.y.z` into `1.0.0`,
  which is a deliberate decision. Raise `y` instead, and propose 1.0.0 separately.
- **Deprecations.** No type covers them, yet SemVer requires MINOR. Use `feat`, so that a
  tool reading the type agrees with SemVer: `feat(cli): deprecate --json`. A `chore` here
  would produce no bump.
- **Unmarked breaking changes.** A commit may break the API without `!`. The diff against
  the public API wins, so the result can be higher than the commits suggest.
- **Other types that change behaviour.** A `perf` or `refactor` commit can still change
  what users observe. The type does not settle that.
- **`feat` outside the public API.** SemVer allows MINOR for substantial private work, but
  does not require it.
- **Work that never shipped.** Compare the release with the last release, not each commit.
  A `fix` for a bug in an unreleased `feat` needs no bump of its own, and a breaking commit
  reverted before release does not count.
- **Reverts.** The specification does not define them, and suggests a `revert` type with a
  footer naming the commits, as in the example above. Reverting something unreleased
  cancels out. Reverting something released is judged like any other change, so removing a
  released public feature is breaking.
- **Automation.** A tool can propose the bump from commits. The cases above are why it
  should not make the decision alone, and a major version still needs the human.
- **Other version schemes.** The table applies to Semantic Versioning. With a date-based
  scheme such as CalVer, commits do not set the number. Keep marking breaking changes
  anyway, because the changelog and any deprecation policy still depend on them.

### Changelog sections

The specification lists generating changelogs as a benefit, while
[Keep a Changelog](https://keepachangelog.com/en/1.1.0/) warns against using commit logs as
changelogs, and defines the sections below. Both hold: commits are the raw material, and the
changelog is curated from them.

| Commit | Changelog section |
|---|---|
| `feat` | Added, or Changed if it extends something existing |
| `feat` that deprecates | Deprecated |
| `fix` | Fixed, or Security if it closes a vulnerability |
| `!` or `BREAKING CHANGE` | Changed or Removed, marked as breaking, with a migration note |
| `perf` | Changed, if users would notice |
| `revert` | Delete the entry if unreleased; otherwise Removed or Fixed |
| `docs`, `style`, `refactor`, `test`, `build`, `ci`, `chore` | Leave out, unless users are affected |

Rewrite the wording for users, and merge related commits into one entry. Do not keep the
type prefixes or hashes as the entry text.

A public commit message can reveal a vulnerability before a fix is released. Follow the
project's disclosure process, and put the detail in the `Security` entry at release time.

## FAQ answers in brief

| Question | Answer |
|---|---|
| What about initial development? | Proceed as if already released. Someone is using the software, and wants to know what is fixed and what breaks. |
| Uppercase or lowercase types? | Any casing, but be consistent. |
| A commit fits more than one type? | Go back and make several commits whenever possible. |
| Does this slow development? | It discourages moving fast in a disorganized way, and helps over the long term. |
| How does it relate to SemVer? | `fix` should become PATCH, `feat` MINOR, and any `BREAKING CHANGE` MAJOR. See [Version bumps](#version-bumps) for the limits of this. |
| I used the wrong type? | Before merging or releasing, edit the history, for example with `git rebase -i`. After release, the cleanup depends on your tools and process. |
| I used a type outside the spec? | It is not the end of the world. Tools based on the spec will just miss that commit. |
| Must all contributors use it? | No. With a squash workflow, lead maintainers can write the final message at merge. |
| How are reverts handled? | Not defined. Tooling authors decide. One recommendation is a `revert` type with a footer referencing the commits. |

## Attribution

Conventional Commits 1.0.0 is published at
[conventionalcommits.org](https://www.conventionalcommits.org/en/v1.0.0/) and licensed under
[CC BY 3.0](https://creativecommons.org/licenses/by/3.0/). This file summarises it in this
skill's own words. The commit examples are this skill's, apart from the `Refs` line in the
`revert` example.
