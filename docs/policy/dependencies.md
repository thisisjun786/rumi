# Dependencies

RUMI follows LINA's [dependency policy](https://github.com/thisisjun786/lina/blob/dev/docs/policy/dependencies.md) unchanged. That policy holds the rules: exact pins, committed `go.mod` and `go.sum`, a pinned toolchain, checksum database verification, `govulncheck`, vendored code kept unchanged with changes in wrapping layers, and one exceptions file. This file states what those rules cover in this repository. They apply from the first change that adds a dependency, and the `dependencies` check enforces them in `foundation` ([CI policy](ci.md)).

## Scope

In this repository, LINA's policy covers:

- Go modules in every `go.mod` and `go.sum`, and the Go toolchain that the `toolchain` line names, used by CI and release builds
- the Flutter toolchain that builds the RUMI app, and the app's pub packages
- the LINA kit and LINA's conformance fixtures, vendored as source from one LINA release tag ([rumi.md](../design/rumi.md))
- LINA's design package, the token source and the shared widgets, vendored from a LINA release tag ([rumi.md](../design/rumi.md))
- pinned binaries, external components such as parsers that need another runtime or license, CI tools, and GitHub Actions
- npm packages that repository tooling needs; repository tooling otherwise uses Node.js built-in modules

Outside this policy are the tools the user has: LINA, the model endpoint and the embedding endpoint the user configures, and opencodex when it serves as that endpoint on LINA OS. RUMI uses them as services and never installs or pins them.

## Clauses for LINA only

The clauses about opencodex and the Node.js runtime that LINA releases ship with it apply only to LINA. RUMI ships no npm package and no Node.js runtime. When repository tooling needs an npm package, LINA's npm rules for repository tooling apply.

## Flutter and pub

LINA's policy gains its rules for the Flutter toolchain and pub packages (the pub lockfile, exact versions, generated Dart types and the generated theme) with the first LINA app implementation. RUMI follows those rules as LINA writes them, for the RUMI app's Flutter toolchain, its pub packages and the vendored design package.

## Vendored from LINA

RUMI vendors three things from LINA release tags, under LINA's vendored code rules, with the release tag as the upstream reference in `vendor.json`:

- **The kit:** the Go module `github.com/thisisjun786/lina/kit`, from the `kit/` directory of the LINA repository. A `replace` entry in RUMI's `go.mod` points the module path to the vendored tree, as LINA's policy allows for a vendored tree in the repository.
- **The conformance fixtures:** `protocol/fixtures/` from the same LINA release tag as the kit. The kit and the fixtures form one vendored tree, so one sync moves both to one tag. `vendor.json` records the content digest of every fixture file.
- **The design package:** the Dart package with the token source and the shared widgets, from the LINA release tag that ships it.

A sync follows LINA's upstream sync steps: one vendored tree per PR, with the upstream range, test results and any change at the wrapper boundary in the PR body.

## The dependencies check

RUMI's `dependencies` check runs the checks in LINA's [CI enforcement table](https://github.com/thisisjun786/lina/blob/dev/docs/policy/dependencies.md#ci-enforcement) that cover what this repository holds: configuration, exact pins, module sums, vulnerabilities, vendored trees, upstream fetch and exceptions; the lockfile, install script and signature checks when repository tooling has an npm package; and the Flutter and pub checks once LINA's policy has them. The script is `.github/scripts/check-dependencies.mjs`, with behavioral tests like the other CI scripts, and exceptions live in `.github/dependency-exceptions.json` under LINA's exceptions rules. The PR that adds the first dependency adds the check, its path mapping and its job ([CI policy](ci.md#go-components)).

The release manifest in [rumi.md](../design/rumi.md#repository-and-releases) records each pinned version and digest.
