# Sandboxing Paseo custom providers with Code on Incus (`coi`)

**Verified against:** `coi` CLI **`v0.16.0`** (source repo inspected at commit `b04fc07` /
`v0.16.0-92-gb04fc07`) and **Paseo `0.15.4`** (`@getpaseo/protocol@0.15.4`,
`@getpaseo/server@0.15.4`) on 2026-11-13.

Two patterns cover almost any agent CLI you want to run inside an ephemeral `coi` Incus container
while keeping native Paseo UI/CLI integration:

1. **Core `extends` provider wrapped by `coi`** (e.g. `claude-coi` extending `"claude"`, where `coi-default` and `coi`'s built-in tool registry already know the agent).
2. **Custom ACP provider wrapped by `coi`** (e.g. `agy-coi` extending `"acp"`, using a custom `coi` image and tool-agnostic `coi run`).

---

## 1. Shared `coi` TOML baseline for Paseo daemon launches

Create a dedicated `coi` TOML file (for example `/home/<user>/.paseo/coi-<id>.toml`) and load it
via `COI_CONFIG` inside your wrapper script so `coi` treats it as **trusted scope**:

```toml
[container]
image = "coi-<id>"               # or "coi-default" if already baked in

[paths]
preserve_workspace_path = true   # keep host workspace paths identical inside the container

[network]
# Avoid interactive sudo nftables prompts when launched non-interactively by the Paseo daemon.
# Use "restricted" or "allowlist" only if passwordless sudoers / network rules are configured.
mode = "open"

[security]
# Set false unless the host `coi` binary has CAP_LINUX_IMMUTABLE; relies on read-only bind mounts.
host_immutable = false
```

In Paseo `~/.paseo/config.json`, set `"paseoTools": { "enabled": false }` unless you explicitly
expose Paseo's host HTTP MCP endpoint (`127.0.0.1:6767`) to the container; otherwise container-side
MCP connections fail or stall.

---

## 2. Pattern 1: Wrapping a core `extends` provider (e.g. `claude-coi`)

When extending a core provider (`claude`, `codex`, `opencode`, `pi`, `omp`) that `coi` already
supports:

1. **Paseo config (`~/.paseo/config.json`):**
   ```json
   "claude-coi": {
     "extends": "claude",
     "label": "Claude (coi)",
     "command": ["/home/<user>/.paseo/bin/claude-coi"],
     "env": { "CLAUDE_CONFIG_DIR": "/home/<user>/.paseo/coi-claude" },
     "paseoTools": { "enabled": false }
   }
   ```
2. **Coi TOML (`~/.paseo/coi-claude.toml`):**
   - Use `[[credentials]] bundle = "<tool>"` for built-in credential seeding.
   - Add a dedicated `[[mounts.default]]` entry for session/project state (e.g. host `~/.paseo/coi-claude/projects` → `/home/code/.claude/projects`) rather than mounting the host agent's full config directory.
3. **Wrapper script caveats:**
   - **Short-circuit metadata probes on the host:** Paseo's `--version` and model-discovery probes (`--no-session-persistence --tools ""` for `claude`) use short deadlines and `SIGKILL`, which can kill `coi run` mid-boot and orphan ephemeral containers. Handle those read-only metadata probes directly; run real sessions inside `coi run -w "$PWD" -- env ...`.
   - **Explicitly forward SDK/API env vars** (`ANTHROPIC_*`, `CLAUDE_CODE_*`, `DISABLE_TELEMETRY`, etc.) into `coi run -- env ...`, while keeping host `CLAUDE_CONFIG_DIR` out of the container so the container writes to `/home/code/.claude`.

---

## 3. Pattern 2: Wrapping an ACP provider with a custom image (e.g. `agy-coi`)

Use `"extends": "acp"` when the provider either isn't in Paseo's `BUILTIN_PROVIDER_IDS` (such as
`antigravity` / `agy`, where `"extends": "antigravity"` is rejected by schema validation) or isn't
built into `coi` (`internal/tool/`).

### A. Build a custom image with the agent installed inside

Avoid bind-mounting host binaries across GLIBC/path boundaries. Build a dedicated image from
`coi-default` (`~/.coi/profiles/<name>/config.toml`):

```toml
[container]
image = "coi-<name>"             # REQUIRED — omitting this silently builds coi-default

[container.build]
base = "coi-default"
commands = [
  "su - code -c 'mise settings add minimum_release_age_excludes <tool>'",
  "su - code -c 'mise use --global <tool>@latest'",
  "ln -sf \"/home/code/.local/share/mise/shims/<bin>\" \"/usr/local/bin/<bin>\"",
]
```

Build with `coi build --profile <name>` (or `--force` to rebuild).

### B. Wire credentials, OAuth state, and env parity in `coi-<name>.toml`

Because `coi` has no `[[credentials]] bundle` for custom agents:

- **OAuth inside containers:** Containers have no desktop GNOME Keyring / D-Bus Secret Service. Mount a dedicated, writable host directory over the agent's file-based secret fallback (e.g. `/home/<user>/.paseo/coi-agy/antigravity-secrets` → `/home/code/.config/antigravity`, `readonly = false`) so OAuth tokens (`plugin-secrets.json`) persist across ephemeral containers.
- **API keys / cloud env:** Declare `[defaults] forward_env = ["GEMINI_API_KEY", ...]` in the trusted `COI_CONFIG` TOML (and/or forward selected vars via `env` inside `coi run`).
- **Dedicated state & cache dirs:** Bind-mount isolated host dirs for config, conversation state, and caches instead of sharing host `~/.gemini` / `~/.config`.
- **No nested sandbox:** Do **not** pass agent flags like `--sandbox` inside `coi`; pass permission-skip flags (`--dangerously-skip-permissions`) if the wrapped ACP server runs unattended.

### C. Lock down ACP `clientCapabilities` in Paseo `config.json`

`GenericACPAgentClient` defaults `clientCapabilities.terminal` to `true`, which lets an ACP server
delegate terminal commands back to the **Paseo host process**. Always turn off host `fs` and
`terminal` capabilities when sandboxing:

```json
"agy-coi": {
  "extends": "acp",
  "label": "Antigravity (coi)",
  "command": ["/home/<user>/.paseo/bin/agy-coi"],
  "options": {
    "clientCapabilities": {
      "fs": { "readTextFile": false, "writeTextFile": false },
      "terminal": false
    }
  },
  "paseoTools": { "enabled": false }
}
```

### D. ACP wrapper mechanics (`cwd`, `--version`, and signal cleanup)

Because Paseo's `ACPAgentClient.spawnTransport` spawns `command` with **`cwd: process.cwd()`**
(the Paseo daemon's working directory) and only passes the target workspace `cwd` later in
`session/new` / `session/load` / `session/resume`:

1. **Fast-path `--version`:** `GenericACPAgentClient.getDiagnostic` runs `[command[0], "--version"]` with a **5 s timeout**. Serve it from a cached version file recorded from the image rather than cold-booting Incus on every diagnostic check.
2. **Resolve `-w <workspace>` from the session JSON-RPC message:** Reply to `initialize` from a cached `acp-init-result.json` (and refresh that cache once the container's `agy acp` replies to the forwarded `initialize`), then inspect `params.cwd` on the first `session/new` / `session/load` / `session/resume` frame to launch:
   `coi run -w <ws> -- [env ...] <agent> acp --dangerously-skip-permissions ...`
3. **Protect `$HOME` during global probes:** Paseo's `fetchCatalog` and `paseo provider diagnostic` pass host `homedir()` as `params.cwd`. Rewrite `cwd === os.homedir()` (or `"/"`) to a dedicated empty directory (e.g. `~/.paseo/coi-<name>/probe-workspace`) before launching `coi run`, so host `$HOME` is never mounted into the container.
4. **Preserve NDJSON stdout & signal teardown:** Keep `coi`'s progress output on `stderr`; forward `SIGINT`/`SIGTERM` to the `coi` child process and wait for exit so `coi` deletes ephemeral containers before terminating.

### E. Authenticate & verify

If using OAuth inside a dedicated container state directory, sign in once inside the isolated
profile before running diagnostics:

```sh
COI_CONFIG=/home/<user>/.paseo/coi-<name>.toml \
  coi run -w /home/<user>/.paseo/coi-<name>/probe-workspace -- <agent> acp --login

paseo reload
paseo provider diagnostic <providerId>
```

Expect `ACP spawn`, `ACP initialize`, `ACP session/new` (listing discovered models and modes), and
`ACP cleanup` to report `ok`.
