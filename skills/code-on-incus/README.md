# Code on Incus (`coi`) skills

[Agent Skills](https://agentskills.io/) for [Code on Incus (`coi`)](https://github.com/coipond/coi):
building custom Incus system container images and running coding agents (including non-built-in
ACP agents) with runtime configuration parity.

This folder is a category, not a skill: it has no `SKILL.md`, and each subfolder is one
self-contained skill.

| Skill | Use it when |
|---|---|
| [coi-custom-containers](coi-custom-containers/SKILL.md) | Defining custom `coi` system container images (`[container.build]` with `base = "coi-default"`), installing agents inside the container image cleanly (including unpinned `mise` installs), and running non-built-in or ACP agents via `coi run` with TOML runtime parity (credentials, state mounts, `forward_env`, workspace paths). |

Verified against `coi` CLI `v0.16.0` and `coi` source commit `b04fc07` (`v0.16.0-92-gb04fc07`)
on 2026-11-13. Where a newer `coi` release or local clone disagrees, the live `coi` binary and
source win.

## Installing

With the [`skills` CLI](https://github.com/vercel-labs/skills):

```sh
npx skills add r3b1s/skills/skills/code-on-incus            # all skills in this category
npx skills add r3b1s/skills --skill coi-custom-containers    # single skill by name
```
