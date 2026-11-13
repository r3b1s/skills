---
name: paseo-custom-providers
description: Create and configure custom agent providers in Paseo (`~/.paseo/config.json`) — extending core providers (`claude`, `codex`, `copilot`, `opencode`, `pi`, `omp`), overriding built-in plugin providers (`antigravity`) in-place, or integrating arbitrary ACP agents with `"extends": "acp"`. Covers config schema validation (`ProviderOverridesSchema`), `command`/`env`/`models`/`additionalModels`/`options` resolution, ACP spawn `cwd` and `clientCapabilities` caveats, probe timeouts, and `paseo reload` / `paseo provider diagnostic`. When sandboxing a Paseo custom provider in a Code on Incus (`coi`) container, read `references/coi-sandboxed-providers.md`.
license: MIT
metadata:
  verified-on: "Paseo CLI/server/protocol 0.15.4 (@getpaseo/protocol@0.15.4, @getpaseo/server@0.15.4 dist), 2026-11-13"
---

# Creating custom providers in Paseo

Custom providers are configured in `~/.paseo/config.json` under `agents.providers.<providerId>`.
Because config validation runs **before** Paseo loads plugins, built-in core providers, plugin-registered
providers, and ACP providers follow distinct rules.

*(If you are wrapping a custom provider inside a Code on Incus (`coi`) container sandbox, read
[references/coi-sandboxed-providers.md](references/coi-sandboxed-providers.md) after reviewing the
core mechanism below.)*

---

## 1. Validation rules & valid `extends` targets

In `@getpaseo/protocol/dist/provider-config.js` (`ProviderOverridesSchema`):

- **Provider ID format:** must match `/^[a-z][a-z0-9-]*$/` (lowercase ASCII, digits, hyphens).
- **Allowed `extends` values (`validExtendsValues`):** strictly the 6 core providers plus `"acp"`:
  - `"claude"`, `"codex"`, `"copilot"`, `"opencode"`, `"pi"`, `"omp"`, `"acp"`.
- **Required fields when `extends` is set on a custom ID:**
  - `label` is **required** (validation fails otherwise).
  - `command` (`string[]`, non-empty) is **required** when `"extends": "acp"`.
- **Plugin providers cannot be extended with `extends`:**
  Built-in plugins (such as `antigravity-provider`, which registers ID `"antigravity"`) load after schema validation. Setting `"extends": "antigravity"` fails validation immediately with `extends unknown provider "antigravity"`. Setting `extends` on an existing plugin's own ID logs `"Plugin provider shadowed by configured provider"` and skips the plugin.

---

## 2. Three customization patterns

### Pattern A: Derive a side-by-side provider from a core `BUILTIN_PROVIDER_IDS` entry

Use when creating a variant (wrapper script, alternate binary/endpoint, custom model list) alongside the default provider:

```json
"claude-custom": {
  "extends": "claude",
  "label": "Claude (Custom)",
  "description": "Custom Claude wrapper",
  "command": ["/abs/path/to/wrapper"],
  "env": { "VAR": "value" },
  "paseoTools": { "enabled": false }
}
```

How `addDerivedProviders` (`@getpaseo/server/.../provider-registry.js`) resolves it:

- `command` becomes `{ mode: "replace", argv }`, replacing the default executable while keeping the base provider's spawn flags, wire protocol, and `options` handling.
- `env` overlays the runtime environment (`CLAUDECODE`, `CLAUDE_CODE_ENTRYPOINT`, `CLAUDE_CODE_SSE_PORT`, `CLAUDE_AGENT_SDK_VERSION` are explicitly cleared by `createProviderEnvSpec` unless set in the overlay).
- `models` (each `{ id, label, description?, isDefault?, thinkingOptions? }`) replaces the discovered catalog; `additionalModels` merges/appends by `id`.
- `paseoTools`: `{ enabled: false }` disables Paseo's local HTTP MCP server injection (`127.0.0.1:6767`), useful if the provider runs isolated or shouldn't access host Paseo MCP tools.

### Pattern B: Override a plugin provider in-place (or write a plugin)

Because plugin IDs (like `"antigravity"`) are not valid `extends` targets:

1. **In-place override (no `extends`):**
   ```json
   "antigravity": {
     "enabled": true,
     "command": ["/abs/path/to/wrapper"],
     "env": { "VAR": "value" }
   }
   ```
   Leaves the ID `"antigravity"` active and switches its launch command to replace mode (`ProviderRuntime.resolveLaunch`). *Caveat:* replaces the native provider rather than adding a second entry, and `antigravity-provider` declares no `options` schema (`ProviderOverrideSchema.options` / `params` are ignored).
2. **Custom plugin:** Write a Paseo plugin calling `server.registerProvider(...)` with a fresh provider ID if you need a plugin-protocol variant to coexist alongside the built-in plugin.

### Pattern C: Arbitrary ACP agent (`"extends": "acp"`)

Any CLI speaking the Agent Client Protocol over stdio NDJSON can be added as a first-class provider:

```json
"my-acp": {
  "extends": "acp",
  "label": "My Agent (ACP)",
  "description": "Custom ACP agent",
  "command": ["/abs/path/to/agent", "acp", "--stdio"],
  "options": {
    "supportsMcpServers": true,
    "clientCapabilities": {
      "fs": { "readTextFile": false, "writeTextFile": false },
      "terminal": false
    }
  }
}
```

Key internals (`providers/generic-acp-agent.js`, `providers/acp-agent.js`, `providers/acp-options.js`):

- **Backing client:** IDs `"cursor"`, `"kimi"`, `"kiro"`, and `"traecli"` instantiate dedicated ACP subclasses; all other IDs instantiate `GenericACPAgentClient`.
- **`clientCapabilities` default:** `BASE_ACP_CLIENT_CAPABILITIES` sets `fs: { readTextFile: false, writeTextFile: false }` and **`terminal: true`**. Set `"terminal": false` in `options.clientCapabilities` if the agent must not execute commands back on the Paseo host.
- **Model, mode, and thinking discovery:** `fetchCatalog` runs `initialize` followed by `session/new` (`mcpServers: []`) with `PROBE_ENV = { NO_BROWSER: "true" }`. Models come from `session/new`'s `models.availableModels` (`currentModelId`) or `configOptions` category `"model"`, modes from `modeState` / `"mode"` config option, and thinking levels from `thought_level` config option. Even when explicit `models` are declared in `config.json`, Paseo still runs the ACP probe to obtain `modes`.
- **Spawn `cwd` vs session `cwd`:** `ACPAgentClient.spawnTransport` launches `command` with **`cwd: process.cwd()`** (the Paseo daemon's CWD), **not** the agent workspace. The real workspace `cwd` is sent over stdin inside `session/new`, `session/load`, and `unstable_resumeSession`. Global catalog probes and `paseo provider diagnostic` send `homedir()` as `params.cwd`.
- **Resume:** Requires the same `cwd` and ACP server support for `loadSession` (`session/load`) or `sessionCapabilities.resume` (`unstable_resumeSession`).

---

## 3. Probes, timeouts & wrapper invariants

If `command` points to a wrapper script rather than a bare CLI binary, respect Paseo's probe lifecycle:

- **Version probe timeout (`5 s`):** `GenericACPAgentClient.getDiagnostic` calls `resolveBinaryVersion` (`[launcher, "--version"]`, or `[runner, ...pkgArgs, "--version"]` for `npx`/`bunx`/`pnpm`/`uvx`) with a **5,000 ms** timeout.
- **ACP diagnostic & close timeouts:** `ACP_DIAGNOSTIC_PHASE_TIMEOUT_MS = 20000` (20 s each for spawn, `initialize`, `session/new`); `ACP_PROBE_CLOSE_TIMEOUT_MS = 2000` and `ACP_CLOSE_REQUEST_TIMEOUT_MS = 2000` (2 s); overall provider refresh deadline `DEFAULT_REFRESH_TIMEOUT_MS = 120000` (overridable via `PASEO_PROVIDER_REFRESH_TIMEOUT_MS`).
- **Core / plugin probe timeouts:** Vary per driver (for example, `antigravity-provider` uses 10 s `PROBE_TIMEOUT` with `SIGKILL` to the process group and 30 s `STARTUP_TIMEOUT`; `claude` uses short metadata probes with `SIGKILL`). Keep `--version` and discovery probes fast and side-effect free.
- **Stdio framing:** ACP parses line-delimited JSON on `stdout` (ignoring non-JSON lines with warnings; stderr is captured into diagnostic/failure output, not streamed live). Other drivers (like `claude` or `antigravity-provider`) may fail fatally on any non-JSON `stdout` line. Keep `stdout` strictly protocol output and send diagnostics/logs to `stderr`.
- **Termination:** Paseo closes stdin / sends `SIGINT`/`SIGTERM` then escalates to `SIGKILL` after a short grace window (2 s in ACP `terminateChildProcess`). Forward signals cleanly to child processes.

---

## 4. Reload & verify

Apply config changes and inspect availability/catalog probes without guessing:

```sh
paseo reload
paseo provider diagnostic <providerId>
paseo provider list
```

If `diagnostic` passes `ACP spawn`, `ACP initialize`, and `ACP session/new` (or core driver version/catalog checks), the provider is ready for `paseo` sessions.
