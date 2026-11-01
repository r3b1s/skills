# Ecosystem differences

Read before choosing a version for a package published to a registry. Each ecosystem's
rules win over plain SemVer for that package. Facts below were read from the linked
documentation on 2026-11-01, and these tools change, so check the current documentation.

For an ecosystem not listed here, find its version specification before choosing a number.
Look for how it treats `0.y.z`, pre-releases, build metadata and a leading `v`.

## npm

[node-semver](https://github.com/npm/node-semver), the package behind npm's version ranges:

- Caret ranges allow changes that keep the left-most non-zero number: `^1.2.3` takes minor
  and patch updates, `^0.2.5` takes patch updates, and `^0.0.4` takes none.
- A range containing a pre-release matches pre-releases only for that same `X.Y.Z`. Other
  ranges skip them unless the option to include pre-releases is set.
- A leading `v` is stripped and ignored.

## Cargo

[Specifying dependencies](https://doc.rust-lang.org/cargo/reference/specifying-dependencies.html):

- Versions are compatible if their left-most non-zero number is the same. Cargo's
  documentation notes that this differs from SemVer, so `0.2.3` means `>=0.2.3, <0.3.0`.
- Version requirements exclude pre-releases unless asked for: if `1.0.0-alpha` is
  published, `foo = "1.0"` does not match it.

## Python

[Version specifiers](https://packaging.python.org/en/latest/specifications/version-specifiers/),
which is not SemVer. Parsing was checked with `packaging` 26.3:

- Only `a`, `b` and `rc` pre-releases exist, with the spellings `alpha`, `beta`, `c`, `pre`
  and `preview` accepted. `1.0.0-rc.1` is accepted and normalizes to `1.0.0rc1`.
- Other pre-release labels are invalid. `1.0.0-canary.1` and `1.0.0-x.7.z.92` are valid
  SemVer and fail here, so use alpha, beta or rc forms in Python projects.
- `+` marks a local version identifier, which should not be used when publishing to a
  public index.
- A leading `v` is ignored, and is omitted from the normalized form.

*Beyond the spec:* in a Python project, use the normalized form in the package metadata
and the changelog, and record which SemVer pre-release it corresponds to if that matters.

## Go

[Go modules reference](https://go.dev/ref/mod):

- Module versions start with `v`, followed by a semantic version.
- From `v2.0.0`, the module path must end in a major version suffix such as `/v2`. A major
  bump therefore changes every import path, and suffixes are not allowed at `v0` or `v1`.
- `v0` versions and pre-releases are unstable and carry no compatibility requirement.
- For a module in a subdirectory, that directory prefixes the tag, for example
  `gopls/v1.2.3`.
- Build metadata is removed when a version is made canonical, except `+incompatible`.

## Several packages in one repository

*Beyond the spec:* give each independently released package its own version and changelog,
and prefix its tags with the package name or path. If everything always releases together,
one version and one changelog is simpler. Choose one approach and record it.
