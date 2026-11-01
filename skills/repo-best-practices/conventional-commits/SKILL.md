---
name: conventional-commits
description: Write and check commit messages with Conventional Commits 1.0.0. Use when adopting Conventional Commits, writing a commit message, choosing a commit type such as feat or fix, marking a breaking change with ! or a BREAKING CHANGE footer, adding footers or trailers, validating a message, reading commits to work out a version bump or draft a changelog, or fixing a mistyped commit.
license: MIT
metadata:
  spec-version: "1.0.0"
  verified-on: "conventionalcommits.org/en/v1.0.0/, read 2026-11-01"
---

# Write Conventional Commits

## Scope

This skill helps a person, an agent, or both write commit messages that state the kind of
change plainly enough for people and tools to act on them.

- **Source:** [Conventional Commits 1.0.0](https://www.conventionalcommits.org/en/v1.0.0/).
  Where this skill and the source disagree, the source wins.
- **Marking:** anything the source does not settle is marked *Beyond the spec*. That is this
  skill's guidance, and a project may decide otherwise.
- **Applies to** any project that has chosen this format. If a project uses another commit
  style, follow that style.
- **Not covered:** choosing a version scheme, and maintaining a changelog. This skill covers
  only how commits feed into them.

## Ground rules

1. **Follow the project's existing style.** Adopting this format is the human's decision.
2. **Describe what the commit really does.** Read the diff first. A correct type on a
   well-formed message is what makes tools trustworthy, and the format cannot check it.
3. **Commit types are evidence.** They suggest a version bump or a changelog entry. They do
   not decide either one, because a breaking change can go unmarked.
4. **Leave published history alone.** Do not rewrite shared or merged commits without
   approval.

## Choose the task

| Task | Go to |
|---|---|
| Set up the format in a project | [Adopt](#adopt) |
| Write a commit message | [Write a commit](#write-a-commit) |
| Use commits for a release | [Review commits for a release](#review-commits-for-a-release) |
| Fix a wrong type or malformed message | [Fix a mistake](#fix-a-mistake) |

## Adopt

1. **Check the history.** See what style `git log --oneline` already shows, and whether any
   tool reads commit messages.
2. **Propose the types.** `feat` and `fix` are the only types with meaning in the
   specification. Other types are the project's choice, and can change over time.
3. **Say so in the contributing guide,** with a link to the specification.
4. **Decide who writes the final message.** Not every contributor has to follow the format:
   with a squash workflow, maintainers can write the final message at merge.
5. *Beyond the spec:* do not rewrite old commits to fit. Start from the next commit.

## Write a commit

```text
<type>[optional scope][optional !]: <description>

[optional body]

[optional footer(s)]
```

1. **Choose the type.** `feat` for a new feature and `fix` for a bug fix; those two are
   required for those cases. Others, such as `docs`, `refactor`, `test` or `chore`, are
   allowed.
2. **Keep one type to a commit.** If a commit fits several, split it whenever possible.
3. **Mark any breaking change, on any type.** Put `!` immediately before the colon, add a
   `BREAKING CHANGE: ` footer, or do both. The footer must be uppercase.
   *Beyond the spec:* use it to say how to migrate.
4. **Mark a deprecation as `feat`.** *Beyond the spec:* no type covers deprecation, and
   Semantic Versioning requires MINOR for it. `feat(cli): deprecate --json` keeps tools in
   agreement.
5. **Add a scope only if it helps.** It is a noun for a part of the codebase, in
   parentheses: `fix(parser): handle empty input`.
6. **Add footers last,** one blank line after the body, as `Token: value` or `Token #value`.
   Tokens use `-` for spaces, so trailers such as `Co-Authored-By:` and `Refs: #123` fit.

Check a message against the
[validation patterns](references/specification.md#validate-a-message), and see
[valid and invalid examples](references/specification.md#examples).

## Review commits for a release

1. **List what is marked.** Run the
   [commands for marked breaking changes](references/specification.md#validate-a-message)
   against the last tag.
2. **Propose the version effect.** With Semantic Versioning, the highest wins across all
   commits since the last release: `fix` points to PATCH, `feat` to MINOR, and `!` or
   `BREAKING CHANGE` on any type to MAJOR. Other types have no effect by themselves. With a
   date-based scheme, commits do not set the number at all.
3. **Check the result against the diff.** *Beyond the spec:* the
   [limits of that mapping](references/specification.md#version-bumps) matter, especially
   for versions below 1.0.0, deprecations, unmarked breaks, reverts, and work that never
   shipped.
4. **Draft the changelog, then curate it.** The specification lists generated changelogs as
   a benefit, while Keep a Changelog warns against using commit logs as changelogs.
   *Beyond the spec:* use commits as a draft, rewrite for users, merge related commits, and
   drop what users cannot see. See
   [how types map to sections](references/specification.md#changelog-sections).

## Fix a mistake

| Situation | What to do |
|---|---|
| Wrong type, not yet merged or released | Edit the history before it lands, with approval if the branch is shared. |
| Wrong type, already merged or released | The cleanup depends on your tools and process. *Beyond the spec:* leave the history alone, and correct any version or changelog by hand. |
| A type outside the specification | It is not the end of the world. Tools based on the specification will miss that commit, so check what they missed. |

## Reference

- [references/specification.md](references/specification.md): the format, rules, valid and
  invalid messages, validation patterns, how commits map to version bumps and changelog
  sections, and the FAQ answers.
