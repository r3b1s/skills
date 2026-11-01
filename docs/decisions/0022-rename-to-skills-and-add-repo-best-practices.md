# 22. Rename the repository to `skills` and add a `repo-best-practices` category

- **Status:** accepted
- **Date:** 2026-11-01
- **Amends:** [0002](0002-scope-of-initial-version.md), [0017](0017-add-qubes-os-skills.md),
  [0018](0018-skill-categories.md), [0020](0020-harness-categories-and-pi-skills.md)

## Context

[0002](0002-scope-of-initial-version.md) scoped this repository to one methodology, and
[0017](0017-add-qubes-os-skills.md), [0020](0020-harness-categories-and-pi-skills.md) and
[0021](0021-add-design-category-and-material-design-3-skill.md) widened it one topic at a
time. It now holds nine skills, most of them unrelated to that methodology, but its name
and the top of its README still described only the methodology.

The owner uses the repository to manage agent skills. When a person or an agent keeps
reimplementing a workflow, that workflow is stored here as a skill to use later. The skills
are for agents and apply to any domain of work; personal skills are not tracked here.

## Decision

- The repository's purpose is the one above. The GitHub repository was renamed from
  `r3b1s/secure-methodologies` to `r3b1s/skills`.
- Add a `skills/repo-best-practices/` category for skills about how a software repository
  is set up, kept and published. It follows [0018](0018-skill-categories.md). A separate
  security category was considered and left out for now.
- Rename the `secure-repo` skill to `repo-security` and move it to
  `skills/repo-best-practices/repo-security/`. This reverses the line in
  [0018](0018-skill-categories.md) that `secure-repo` stays at `skills/<skill>/`. Only the
  skill's name and location change.
- The root README describes the whole collection. The methodology's scope and foundation
  move to the [category README](../../skills/repo-best-practices/README.md), so the
  statements in [0002](0002-scope-of-initial-version.md),
  [0017](0017-add-qubes-os-skills.md) and [0020](0020-harness-categories-and-pi-skills.md)
  about "the scope in the README" now refer to that file.
- Baseline maintenance guidance moves from `docs/maintenance/` to
  `docs/baseline-maintenance/`, a subcategory of the repository-wide docs.

## Consequences

- The methodology installs with `npx skills add r3b1s/skills --skill repo-security`. Copies
  installed as `secure-repo` keep that name until replaced, and `--skill secure-repo` no
  longer finds anything.
- GitHub redirects the old repository URL, so earlier commands that use only the old
  repository name keep working as long as `r3b1s/secure-methodologies` is not created
  again.
- In earlier records, links and file paths that would break were updated. Their prose,
  including `secure-repo` and the install commands in the consequences of
  [0018](0018-skill-categories.md), [0020](0020-harness-categories-and-pi-skills.md) and
  [0021](0021-add-design-category-and-material-design-3-skill.md), stays as written, as a
  record of the time.
- Further repository practices can be added under `skills/repo-best-practices/` without a
  new category record.
