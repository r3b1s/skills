# skills

[Agent Skills](https://agentskills.io/) for agents to reuse across any domain of work. When
a person or an agent keeps reimplementing a workflow, it is stored here as a skill, so the
next agent can load it instead of working it out again.

Every skill is self-contained, rests on primary sources, and says what it was checked
against.

## Skills

| Category | Skills | Covers |
|---|---|---|
| [repo-best-practices](skills/repo-best-practices/) | `repo-security`, `semver`, `calver`, `conventional-commits`, `keep-a-changelog` | How a software repository is set up, kept and published: making it safe to publish and easy to trust, versioning it, and recording what changed. |
| [qubes-os](skills/qubes-os/) | `qubes-dom0-cli`, `qubes-vm-cli`, `qubes-disposable-vms`, `qubes-admin-python` | The dom0 and in-qube command-line tools, building disposable qubes, and the `qubesadmin` Python library. |
| [harness-pi](skills/harness-pi/) | `pi-monorepo-info`, `pi-core`, `pi-durable` | The pi coding-agent harness: its monorepo, the regular harness internals, and the experimental `pi-durable` runtime. |
| [design](skills/design/) | `material-design-3` | UI/UX design systems, starting with Material Design 3 and M3 Expressive. |

Related skills sit together as `skills/<category>/<skill>/`, and skills specific to one
agent harness go in a `harness-<name>` folder. Category folders hold skills but are not
skills themselves; see [decision 0018](docs/decisions/0018-skill-categories.md) and
[decision 0020](docs/decisions/0020-harness-categories-and-pi-skills.md).

## Installing

The skills follow the [Agent Skills](https://agentskills.io/specification) format and are
found by the [`skills` CLI](https://github.com/vercel-labs/skills) in the `skills/` folder,
including category folders such as `skills/qubes-os/`.

```sh
npx skills add r3b1s/skills --list                  # see what is available
npx skills add r3b1s/skills --skill repo-security   # one skill, by name
npx skills add r3b1s/skills/skills/qubes-os         # a whole category
```

Add `-g` to install for your user rather than the current project. Every skill is
self-contained: it needs nothing else from this repository.

## Repository best practices skills

[skills/repo-best-practices/](skills/repo-best-practices/) holds skills for how a software
repository is set up, kept and published. `repo-security` makes a repository safe to publish
and easy to trust, through the audit before it goes public, a new project's start, ongoing
maintenance, releases and periodic re-audits. Its requirements come from the
[OpenSSF OSPS Baseline](https://baseline.openssf.org/), and it works for a person following
it alone as well as for an agent. Its full scope and foundation are in
[the category README](skills/repo-best-practices/README.md).

The category also has one skill per published standard, each pinned to one version of it:
`semver` for [Semantic Versioning 2.0.0](https://semver.org/spec/v2.0.0.html), `calver` for
[Calendar Versioning](https://calver.org/), `conventional-commits` for
[Conventional Commits 1.0.0](https://www.conventionalcommits.org/en/v1.0.0/), and
`keep-a-changelog` for [Keep a Changelog 1.1.0](https://keepachangelog.com/en/1.1.0/). Each
works alone, and guidance that goes beyond its standard is marked as such. See
[decision 0023](docs/decisions/0023-one-skill-per-standard.md).

```sh
npx skills add r3b1s/skills/skills/repo-best-practices    # all of them
npx skills add r3b1s/skills --skill semver                # just one
```

## Qubes OS skills

[skills/qubes-os/](skills/qubes-os/) holds skills for
[Qubes OS](https://www.qubes-os.org/): which command-line tools exist in dom0
(`qubes-dom0-cli`) and inside other qubes (`qubes-vm-cli`), and how to build disposable
qubes whose configuration and secrets persist only where intended
(`qubes-disposable-vms`), and how to manage qubes from Python with `qubesadmin`, the
library over the Admin API (`qubes-admin-python`). Their scope is using Qubes OS and
building its qubes correctly. They were checked against Qubes OS R4.3.2 and the upstream
documentation; `qubes-admin-python` against the R4.3 source and an offline test fake.

```sh
npx skills add r3b1s/skills/skills/qubes-os          # all of them
npx skills add r3b1s/skills --skill qubes-dom0-cli    # just one
```

## pi harness skills

[skills/harness-pi/](skills/harness-pi/) holds skills for developing with and on
[pi](https://github.com/earendil-works/pi), a minimal coding-agent harness:
`pi-monorepo-info` (the official monorepo: packages, dependency graph, build and check
scripts, contributor rules, releases), `pi-core` (the regular harness: provider layer,
agent loop, sessions, tools, extensions, supporting libraries) and `pi-durable` (the
experimental durable runtime and how it differs). They were read from the pi source at
commit `74f468640` (packages 1.4.0), not by running pi, and pi changes quickly, so a clone
of pi wins over them.

```sh
npx skills add r3b1s/skills/skills/harness-pi    # all of them
npx skills add r3b1s/skills --skill pi-core       # just one
```

## Design skills

[skills/design/](skills/design/) holds UI/UX design-system skills, starting with
`material-design-3` for [Material Design 3](https://m3.material.io/) and **Material 3
Expressive**: the three-tier token architecture, HCT dynamic color and surface-container
elevation, spring-based motion physics, the 30-style baseline + emphasized type scale, the
10-step corner shape scale and shape morphing, adaptive window size classes, component
selection (including M3 Expressive button groups, split buttons, floating toolbars, FAB
menus, and loading indicators), official upstream repositories, and incremental adoption in
existing Web, Desktop/Studio, Android, and Flutter projects. See
[decision 0021](docs/decisions/0021-add-design-category-and-material-design-3-skill.md).

```sh
npx skills add r3b1s/skills/skills/design              # all of them
npx skills add r3b1s/skills --skill material-design-3  # just one
```

## Documentation

| Path | What it is |
|---|---|
| [docs/decisions/](docs/decisions/) | Decision records: what was decided, when and why. |
| [docs/research/](docs/research/) | Dated research behind the decisions. |
| [docs/baseline-maintenance/](docs/baseline-maintenance/) | Maintainer guidance for moving `repo-security` to a new OSPS Baseline version. |
| [docs/glossary.md](docs/glossary.md) | Terms as this repository uses them, with their sources. |

## Licence

[MIT](LICENSE).
