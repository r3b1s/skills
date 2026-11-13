# Paseo harness skills

[Agent Skills](https://agentskills.io/) for configuring, extending, and wrapping providers in
[Paseo](https://getpaseo.com/).

This folder is a category, not a skill: it has no `SKILL.md`, and each subfolder is one
self-contained skill.

| Skill | Use it when |
|---|---|
| [paseo-custom-providers](paseo-custom-providers/SKILL.md) | Adding or customizing providers in `~/.paseo/config.json`: extending built-in core providers, overriding plugin-registered providers in-place, wiring arbitrary ACP agents via `extends: "acp"`, configuring `options`/`models`/`env`, or debugging probes, timeouts, and `paseo provider diagnostic`. Includes `references/coi-sandboxed-providers.md` for sandboxing Paseo providers with `coi`. |

Verified against installed Paseo CLI/server/protocol `0.15.4` (and `coi` CLI `v0.16.0` / source
commit `b04fc07` in the `coi` reference) on 2026-11-13.

## Installing

With the [`skills` CLI](https://github.com/vercel-labs/skills):

```sh
npx skills add r3b1s/skills/skills/harness-paseo             # all skills in this category
npx skills add r3b1s/skills --skill paseo-custom-providers   # single skill by name
```
