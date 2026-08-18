# Repository Guidelines

## Project Overview

This repository packages upstream [`kahme247/ompweb`](https://github.com/kahme247/ompweb) for Arch Linux/AUR. It contains packaging and release automation only—not the Next.js application source. `ompweb` is a local web UI for the `oh-my-pi` coding agent.

Application behavior belongs upstream. Changes here should normally affect the package recipe, generated AUR metadata, release workflow, or packaging documentation.

## Architecture & Data Flow

The repository is a thin, immutable-artifact packaging pipeline:

1. `.github/workflows/aur-publish.yml` resolves the latest stable upstream GitHub release and matching npm package.
2. CI verifies npm SHA-512 integrity, fetches the same-tag `package-lock.json`, and runs `npm ci --omit=dev --ignore-scripts`.
3. CI builds the Linux x86_64/glibc runtime archive twice with normalized tar/gzip metadata and requires byte-identical output.
4. The archive is published under a SHA-256-addressed immutable GitHub Release tag.
5. CI updates `PKGBUILD`, regenerates `.SRCINFO`, and builds the package without network access.
6. Package layout assertions and an HTTP startup smoke check run before recipe changes are committed and mirrored to AUR.

At runtime, `PKGBUILD` installs the bundle under `/usr/lib/node_modules/@kahme247/ompweb` and creates `/usr/bin/ompweb` as a relative symlink to `bin/omp-web.js`. The CLI starts the loopback web UI and invokes the installed `omp` command; `OMP_WEB_OMP_BIN` can override that binary.

There is no local dependency-injection, state-management, request-handling, or application async architecture to preserve. Inspect or modify the upstream repository for those concerns.

## Key Directories

- `.github/workflows/` — release discovery, deterministic bundle creation, package validation, GitHub release publication, and AUR synchronization.
- `src/` — ignored `makepkg` source workspace; generated locally, not application source.
- `pkg/` — ignored `makepkg` package staging tree.

No tracked `src`, `app`, `packages`, `tests`, or scripts directory exists.

## Development Commands

Run package operations on Arch Linux or in an `archlinux:base-devel` container:

```bash
# Verify the checksum-addressed source archive
makepkg --verifysource --force --noconfirm

# Build and install locally with declared dependencies
makepkg -si

# Regenerate committed AUR metadata after changing PKGBUILD
makepkg --printsrcinfo > .SRCINFO

# Exercise the installed entry point
ompweb --no-open --port 30177
```

The CI package build uses `makepkg --force --nodeps --noconfirm` only inside a controlled container after source verification. Do not copy that dependency-skipping flag into normal installation instructions.

There are no repository-local npm scripts, build aliases, lint commands, formatter commands, or type-check commands. Bundle construction is implemented inline in `.github/workflows/aur-publish.yml` rather than as a local script.

## Code Conventions & Common Patterns

- Follow standard `PKGBUILD` Bash conventions: two-space indentation, quoted expansions, arrays for package metadata, and lowercase snake_case locals/functions.
- Use uppercase snake_case for workflow environment variables and step inputs/outputs, such as `PACKAGE_NAME`, `VERSION`, and `BUNDLE_SHA256`.
- Start nontrivial workflow shell blocks with `set -euo pipefail`; validate prerequisites explicitly and exit nonzero with a useful stderr message.
- Preserve the exact single-line forms consumed by the workflow updater:

  ```bash
  pkgver=VERSION
  pkgrel=INTEGER
  _bundle_sha256='64-lowercase-hex-characters'
  ```

  Embedded Python regex assertions depend on these shapes. A version change resets `pkgrel` to `1`; a checksum-only change increments it.
- Treat `.SRCINFO` as generated output from `PKGBUILD`; never update it independently.
- Preserve release invariants unless intentionally redesigning the pipeline: same-tag lockfile, npm SHA-512 verification, disabled lifecycle scripts, deterministic double-build, immutable SHA-256 release URL, offline package build, and pinned AUR SSH host verification.
- Keep repository-owned comments and documentation in English.
- No application-level naming, error-handling, async, dependency-injection, or state-management convention can be inferred from this packaging-only checkout.

## Important Files

- `PKGBUILD` — canonical editable package recipe, dependencies, checksum-pinned source, and installed filesystem layout.
- `.SRCINFO` — committed generated AUR metadata; must match `PKGBUILD`.
- `.github/workflows/aur-publish.yml` — complete update, reproducibility, validation, release, commit, and AUR publication pipeline.
- `README.md` — package purpose, installation, runtime usage/security notes, layout, and release workflow.
- `.gitignore` — excludes `makepkg` work trees and generated archives/packages.

The installed application entry point is `bin/omp-web.js` inside the external runtime bundle; it is not tracked here.

## Runtime/Tooling Preferences

- Runtime: Node.js `>=22.19.0` plus the `oh-my-pi` package.
- Supported package target: Linux x86_64 with glibc.
- Arch packaging tool: `makepkg`; CI uses `archlinux:base-devel` containers.
- JavaScript package manager: npm, used only by release automation with the upstream `package-lock.json`. Do not substitute Bun, pnpm, or Yarn.
- CI tooling also relies on Bash, GitHub CLI (`gh`), `jq`, `curl`, OpenSSL, GNU tar/gzip, Python 3, Docker, Git, and SSH tools.
- `AUR_SSH_PRIVATE_KEY` is the only referenced repository secret. Keep secrets out of files and logs.

## Testing & QA

No unit, integration, or end-to-end test suite, test framework, coverage tool, `check()` function, or coverage threshold exists in this repository. QA is package-oriented and lives in `.github/workflows/aur-publish.yml`:

- verify upstream npm integrity and manifest version;
- build the normalized bundle twice and compare bytes;
- run `makepkg --verifysource` and build with networking disabled;
- assert the executable symlink, license, lockfile, and `.next` output exist;
- start the staged `/usr/bin/ompweb` with `--no-open --port 30177`;
- poll `http://127.0.0.1:30177/` and require `<title>omp web</title>`.

For recipe changes, regenerate `.SRCINFO`, verify the source, build the package, and repeat the installed-entry-point smoke scenario. Application behavior changes and their tests belong in the upstream application repository.
