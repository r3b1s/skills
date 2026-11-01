# 23. Keep one skill per standard

- **Status:** accepted
- **Date:** 2026-11-01

## Context

Semantic Versioning, Conventional Commits and Keep a Changelog were first written up as one
draft skill, `repo-versioning`, which was never committed. They are separate standards, with
separate versions, and a project can adopt any of them without the others. Calendar
Versioning is a fourth, and an alternative to Semantic Versioning.

## Decision

- A published standard gets its own skill, named for the standard. Skills are not bundled
  because they tend to be used together.
- `skills/repo-best-practices/` holds `semver`, `calver`, `conventional-commits` and
  `keep-a-changelog`, each pinned to one version of its standard in its `metadata`.
- Where two standards meet, each skill states what it needs in its own words
  ([0010](0010-skills-self-contained.md)). It may name another standard and link to that
  standard's own site, but it does not link to another skill. For example, deciding what a
  commit means for a version number belongs in `conventional-commits`, and `semver` treats
  commit types only as evidence.
- [calver.org](https://calver.org/) is licensed under CC BY-SA 4.0, while this repository is
  MIT ([0008](0008-mit-licence.md)). The `calver` skill therefore records notation and facts
  in its own words, and does not adapt the site's text.

## Consequences

- Each standard can be installed, updated and pinned on its own.
- A release that uses several standards takes several skills, and no single skill describes
  the whole flow.
- Some guidance appears in more than one skill, such as the rule not to change a released
  version, and must be updated in each.
