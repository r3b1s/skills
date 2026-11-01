# Ecosystem limits on CalVer

Read before proposing a scheme for anything published to a registry. Each ecosystem's rules
win. Facts below were read from the linked documentation on 2026-11-01, and tested where
noted, but these tools change, so check the current documentation.

For an ecosystem not listed here, find out whether it requires SemVer, how it treats leading
zeroes, how many segments it allows, and whether it reads the first number as a compatibility
boundary.

## At a glance

| Target | Padded | Two segments | Four segments | Good scheme |
|---|---|---|---|---|
| npm | no | no | no | `YYYY.M.MICRO` |
| Cargo | no | see below | see below | `YYYY.M.MICRO` |
| Go modules | no | no | no | prefer SemVer |
| Python | accepted, then normalized | yes | yes | `YYYY.M.MICRO` or `YYYY.M.D` |
| Linux packages | accepted, compared numerically | yes | yes | any |
| Container tags, filenames | useful | yes | yes | `YY.0M` or `YYYY.0M.0D` |

## npm

Tested with the `semver` package on 2026-11-01:

- `2026.4.1` and `2026.8.19` are valid.
- `2026.4` is invalid: three numbers are required.
- `2026.04.1` and `26.04` are invalid: leading zeroes are not allowed.
- `2026.9.11.1` is invalid: four numbers are not allowed.

A caret range stops at the next first number, so `^26.4.0` accepts `26.11.0` but not
`27.1.0`. *Beyond the spec:* with a year as the first number, `^` ranges do not cross a new
year, so tell users how to stay current.

## Cargo

calver.org states that Cargo enforces strict SemVer, so do not pad. Not tested here.
[Cargo's documentation](https://doc.rust-lang.org/cargo/reference/specifying-dependencies.html)
treats versions as compatible when their left-most non-zero number matches, so a new year's
first number reads as an incompatible release. Use three unpadded numbers.

## Go modules

calver.org states that Go enforces strict SemVer, so do not pad. Not tested here.

The [Go modules reference](https://go.dev/ref/mod) requires a module path suffix matching the
major version from `v2` onward. *Beyond the spec:* a year as the first number would need a
path such as `example.com/mod/v2026`, changing every import path each year. Prefer SemVer for
Go libraries.

## Python

[Version specifiers](https://packaging.python.org/en/latest/specifications/version-specifiers/).
Tested with `packaging` 26.3 on 2026-11-01:

- Leading zeroes are normalized away: `2026.08.19` becomes `2026.8.19`, and `26.04` becomes
  `26.4`. PyPI shows the normalized form, so padding gains nothing there, and the tag and the
  listed version can differ.
- Four numeric segments are valid: `2026.9.11.1`.
- Pre-releases use `a`, `b` or `rc`: `2026.9.0rc1`.
- Version epochs let a new scheme sort above an old one: `1!26.4.0` ranks above `30.2.0`.

## Linux packages

calver.org states that dpkg and rpm compare numerically, so `04` and `4` are the same number.
Not tested here.

## Container tags and filenames

These often sort as plain text, which is where padding helps. Tested with `sort` on
2026-11-01:

- Padded: `26.04`, `26.09`, `26.10` sort correctly as text.
- Unpadded: `26.10`, `26.4`, `26.9` is the text order, which is wrong.
- `sort -V` orders the unpadded form correctly: `26.4`, `26.9`, `26.10`.
