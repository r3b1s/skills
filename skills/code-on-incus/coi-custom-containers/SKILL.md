---
name: coi-custom-containers
description: Define custom Incus system container images for Coi (Code on Incus) and wire non-built-in coding agents — such as ACP-extending agents or custom harnesses — inside those images using tool-agnostic `coi run` with explicit TOML runtime parity. Use when building `[container.build]` images based on `coi-default`, installing agent CLIs inside containers rather than bind-mounting host binaries, managing unpinned `mise` installs cleanly inside images, or wiring `[[credentials]]`, `[[mounts.default]]`, `forward_env`, and workspace path preservation for non-built-in tools.
license: MIT
metadata:
  verified-on: "coi CLI v0.16.0 (runtime) and coi source b04fc07 / v0.16.0-92-gb04fc07 (2026-11-13)"
---

# `coi` custom containers and non-built-in agents

Code on Incus (`coi`) has built-in registry integrations (`internal/tool/`) only for a fixed set
of agents (`claude`, `codex`, `opencode`, `pi`, `omp`, etc.). Any other agent — such as an
ACP-compatible agent CLI or custom harness — still works through `coi` by:

1. **Installing the agent inside a custom `coi` system container image** derived from `coi-default`, rather than bind-mounting host binaries.
2. **Launching it via tool-agnostic `coi run -- <cmd>`** backed by an explicit `coi` TOML file that mirrors built-in tool conveniences (credentials, persistent state directories, env vars, and workspace path mapping).

---

## 1. Custom system container images (`[container]` + `[container.build]`)

### Profile layout & discovery

`coi build --profile <name>` scans `profiles/<name>/config.toml` under three roots (`GetProfileParentDirs`):

1. `~/.coi/profiles/<name>/config.toml` (trusted scope)
2. `dirname($COI_CONFIG)/profiles/<name>/config.toml` (trusted scope when `COI_CONFIG` is set)
3. `./.coi/profiles/<name>/config.toml` (relative to the current working directory; untrusted scope)

Duplicate profile names across roots are an error. `coi build --all` builds every visible profile with `[container.build]`; `coi build --profile <name> --force` rebuilds an existing alias.

### Critical naming rules (avoid clobbering `coi-default`)

- **Always set `[container] image = "<custom-alias>"` alongside `[container.build]`.**
  If `[container] image` is omitted, `buildCommand` defaults `imageName` to `"coi-default"` and silently ignores `[container.build]` `commands` and `script`, running the built-in default builder instead.
- **Avoid alias prefixes starting with `coi-default`** (such as `coi-default-foo`): image version pruning/listing matches by prefix (`ListVersions`).
- **`[container] alias` is not the image name:** `alias` labels running session containers (`user.coi.alias`); `image` selects/names the Incus image alias.

### How custom builds execute

When `[container] image` is a custom name and `[container.build]` sets `commands` or `script` (`script` takes precedence if both exist):

- `base` defaults to `"coi-default"` (must already exist locally).
- `coi` launches a fixed temporary container named `coi-build` (so do not run two `coi build` invocations concurrently) with open-mode outbound internet (`EnsureOpenModeRules`).
- The build script is pushed to `/tmp/build.sh` and executed **as `root`** with **no `COI_*` env vars**. Host files adjacent to `script` are not copied into the container — inline all assets or fetch them inside the script.
- What `coi-default` already provides (verified on `v0.16.0` / `ubuntu:24.04` base): `code` user (UID 1000, passwordless `sudo`), system Node.js 22 (`node`, `npm` under `/usr/bin`), `/usr/local/bin/mise`, `git`, `curl`, `wget`, `jq`, `ripgrep`, `tmux`, `docker`, `gh`.
- After the script exits 0, `coi` stops `coi-build`, publishes `<image>-YYYYMMDD-HHMMSS`, and repoints the `<image>` alias (alternative ad-hoc workflow: `coi image publish <stopped-container> <alias>`).

### Pattern: install agents inside the container with `mise` (unpinned `@latest`)

Prefer installing inside `[container.build]` over host bind-mounts so images are portable and self-contained without host GLIBC or toolchain coupling:

```toml
# ~/.coi/profiles/<agent>/config.toml
[container]
image = "coi-<agent>"

[container.build]
base = "coi-default"
commands = [
  # Bypass mise minimum_release_age filters before resolving @latest so brand-new releases install:
  "su - code -c 'mise settings add minimum_release_age_excludes <tool-id> && mise settings add minimum_release_age_excludes npm:<pkg>'",
  # Install globally as the container `code` user; omit `--pin` so config.toml stays `"latest"`:
  "su - code -c 'mise use --global <tool-id>@latest npm:<pkg>@latest'",
  # Expose mise shims on default PATH inside the container for non-interactive `coi run` (`incus exec`):
  "for bin in <bin1> <bin2>; do ln -sf \"/home/code/.local/share/mise/shims/$bin\" \"/usr/local/bin/$bin\"; done",
  "su - code -c '/usr/local/bin/<bin1> --version'",
]
```

Key mechanics:

- **Run `mise` as `code` (`su - code -c '...'`):** `commands` execute as `root`, whereas `coi run` sessions execute as user `code` (`/home/code`), whose global config lives at `/home/code/.config/mise/config.toml`.
- **Keep versions unpinned:** `mise use --global <tool>@latest` (without `--pin`) records `"<tool>" = "latest"` in `/home/code/.config/mise/config.toml`. Setting `minimum_release_age_excludes` prevents fresh upstream releases from being hidden during image builds.
- **Symlink shims inside the container, never on the host:** `coi run` executes non-interactive `incus exec` without `.bashrc` (`eval "$(mise activate bash)"`). Symlinking `/usr/local/bin/<bin> -> /home/code/.local/share/mise/shims/<bin>` *inside the build script* puts the binary on system `PATH`, preserves `argv[0]` shim dispatch, and tracks future in-container `mise upgrade` installs without touching host `/usr/local/bin`.

Verify after build:

```sh
coi build --profile <agent>
COI_CONFIG=/path/to/coi-<agent>.toml coi run -w /tmp -- sh -c 'cat /home/code/.config/mise/config.toml && <bin1> --version'
```

---

## 2. Runtime parity via `coi run` TOML config

`coi run -- <binary> [args...]` works for any binary, bypassing `internal/tool/`. Use a dedicated config file loaded via `COI_CONFIG=/path/to/coi-<name>.toml` (which gives **trusted scope**, required for `forward_env` and arbitrary host mounts).

### What built-in `Tool` conveniences CAN be wired in TOML

| Built-in convenience | Explicit TOML parity for non-built-in agents under `coi run` |
|---|---|
| Credential bundle seeding (`[[credentials]] bundle = "..."`) | Use ad-hoc `[[credentials]]` entries (`host`, `container`, `mode = "0600"`). Calls the same `seedHostFile` helper (`mkdir -p`, push, `chown` to `code`, `chmod`). Missing host files log and skip cleanly; refreshes on persistent container reuse. |
| OAuth / mutable token & state persistence | Prefer writable `[[mounts.default]]` (`host = ...`, `container = "/home/code/..."`, `readonly = false`) backed by a **dedicated host directory** (e.g. `~/.config/<app>` or `coi-<name>/secrets`). Mutable mounts persist OAuth refresh tokens across ephemeral containers; `[[credentials]]` is one-way copy only. |
| Separate container conversation/cache dirs | Wire dedicated `[[mounts.default]]` dirs rather than sharing host `~/.<agent>/` state (avoids mixing host conversation DBs or host-only `trustedWorkspaces`). |
| API-key / env-based auth | `[defaults]` `forward_env = [...]` (trusted `COI_CONFIG` scope only), `[defaults.environment]`, or `[defaults.env_commands]` (`env_command_timeout = "30s"`). |
| Host workspace path matching flags/JSON-RPC `cwd` | `[paths] preserve_workspace_path = true` mounts the workspace at the identical host path inside the container. |
| Git, security, network, limits, timezone | `[git]` (`strip_attribution`, `protected_branches`), `[security]` (`host_immutable = false` if host lacks `CAP_LINUX_IMMUTABLE`, `protected_paths`, `secret_paths`), `[network]` (`mode = "open"` when daemon lacks passwordless `sudo` for `nft`), and `[limits]` all apply identically in `coi run`. |

### What CANNOT be wired via TOML under `coi run`

Because `coi shell` (`session.Setup` / `setup_phase_tooling.go`) owns `Tool`-registry hooks that `phases_run.go` does not run for generic commands:

- **Automatic permission-bypass settings merge (`GetSandboxSettings()`):** Pass the agent's headless/skip-permissions flags in the `coi run -- <agent> ...` command line, or seed a static settings file via `[[credentials]]` / `[[mounts.default]]`.
- **Sandbox context files (`[tool] auto_context`, `~/SANDBOX_CONTEXT.{md,json}`):** Injected by `coi shell` only.
- **`[tool] pre_launch` and `[tool]` model/effort env helpers:** `coi shell` only; bake setup into `[container.build]` or the launcher script.
- **`coi run --prompt` / `DiscoverSessionID`:** Restricted to built-in `claude` and `ToolWithHeadlessPrompt`; external harnesses pass session IDs/prompts over CLI args or ACP stdio directly.

---

## 3. Additional requirements for ACP (`--stdio`) & non-interactive wrappers

When wrapping an agent that runs as an ACP stdio server (or headless NDJSON worker) inside `coi run`:

1. **Keyring-less OAuth inside containers:** Linux Incus containers lack desktop GNOME Keyring / D-Bus Secret Service. Verify the agent's file-based token fallback path and bind-mount that config directory (`readonly = false`) from a dedicated host state directory so OAuth login (`<agent> acp --login` or similar) persists across ephemeral containers. Keep `[defaults] forward_env` populated too if API keys may also be used.
2. **Never enable nested sandboxing inside `coi`:** Omit agent flags such as `--sandbox` (bubblewrap/Seatbelt inside Incus commonly fails or weakens nothing useful when the whole container is the boundary).
3. **Keep `stdout` pure JSON / NDJSON:** `coi run` writes container setup, mount, and cleanup logs to `stderr`. Never redirect `stderr` into `stdout`, and suppress interactive banners/spinners in the agent invocation.
4. **Prevent `cwd` mismatch & accidental `$HOME` mounts:** If the caller (for example a daemon spawning an ACP server) invokes the wrapper with `cwd = process.cwd()` and only supplies the real workspace path inside an ACP `session/new` / `session/load` / `session/resume` JSON-RPC payload, either resolve `-w` from that message before starting `coi run`, or map diagnostic probes targeting host `$HOME` to a dedicated empty `probe-workspace` directory so host `$HOME` is never mounted into the container.
5. **Fast metadata probes & clean signal cleanup:** Handle quick version probes (`--version`) without paying a cold container boot if the caller enforces a short timeout (e.g. `<= 5s` with `SIGKILL`, which would otherwise orphan ephemeral containers). When wrapping `coi run` from a helper process, forward `SIGINT` and `SIGTERM` directly to the `coi` child and wait for it to delete ephemeral containers before exiting.
