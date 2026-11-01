# Public API and breaking changes

Read when declaring a public API or deciding whether a change is breaking.

SemVer requires a public API (item 1), but says only that it may be declared in code or
documentation, and should be precise and comprehensive. It does not list what belongs in
one, so everything below is *Beyond the spec*: this skill's guidance for a human to confirm.

## Decide what is public

The test is whether someone outside the project can reasonably depend on it. Declare the
answer in writing, because a declared boundary is what turns later arguments into quick
checks.

| Project type | Usually public | Usually not |
|---|---|---|
| Library | exported functions, types, classes and constants, their documented behaviour and errors, minimum supported runtime | unexported or `internal` code, undocumented behaviour |
| CLI | commands, flags, exit codes, machine-readable output, config file format, environment variables | human-readable message wording, log text |
| HTTP or RPC service | routes, request and response schemas, status codes, error formats, authentication | internal services, response timing |
| Plugin host | hooks, extension interfaces, event payloads | host internals |
| File or data format | the format's fields and their meaning, and which older versions can still be read | how it is produced |
| Application with no integrators | stored user data and its upgrade path, documented behaviour | everything else |

Say what is excluded as well as what is included. A short statement in the README is enough:

```markdown
## Versioning

This project follows [Semantic Versioning](https://semver.org/spec/v2.0.0.html). The public
API is the `acme` command's subcommands, flags, exit codes and `--format json` output.
Human-readable output and anything under `internal/` can change in any release. See the
[changelog](CHANGELOG.md).
```

## Judge a change

Ask whether code or a workflow that works today, using the API as documented, stops working
or changes meaning after an upgrade with no edits. If so, it is breaking.

| Change | Result |
|---|---|
| Remove or rename anything public | breaking |
| Add a required parameter, field or flag | breaking |
| Add an optional parameter, field or flag with the old behaviour as default | compatible |
| Change a default value | breaking if results change |
| Reject input that used to be accepted | breaking, unless the input was documented as invalid |
| Accept input that used to be rejected | compatible |
| Change a return type, output format or exit code | breaking |
| Remove a value from a response | breaking |
| Add a field to a response | compatible if consumers are expected to ignore unknown fields; say so in the API statement |
| Change a data format without reading the old one | breaking |
| Drop a supported runtime, platform or dependency range | treat as breaking if the declaration includes them |
| Make something much slower, or raise resource needs a lot | judgment; breaking if documented limits no longer hold |
| Fix a bug people rely on | judgment; see the [FAQ answer](specification.md#faq-answers-in-brief) |

When it is unclear, ask who would be affected and how badly, then give the human the
tradeoff. A number that overstates a change costs an upgrade review. A number that
understates it breaks someone's build.

Check for accidental changes as well as intended ones. Compare the public surface against
the last release, for example the exported symbols, CLI help output, or API schema.
