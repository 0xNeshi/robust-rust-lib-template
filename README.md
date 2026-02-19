# Robust Rust Library Template

This repository is a **Rust library project template** intended to establish a
**quality baseline** for new Rust crates (and to act as a reference checklist
for improving existing ones).

It is a practical baseline for the proposal **“Rust Project Template:
Establishing a Quality Baseline”**:

- **Minimal by default**: no dependencies are included unless you add them.
- **Policy + automation**: MSRV and dependency-version discipline is encouraged
  and enforced via CI.
- **“Complete project” defaults**: CI, formatting/linting, security policy,
  contribution & governance scaffolding, issue/PR templates.

## How to use this template

### 1) Create a new repository from it (GitHub template)

You can instantiate a new crate by using GitHub’s **“Use this template”** button.

After creation:

- **Rename placeholders** in `Cargo.toml` and other files (see
  [File-by-file customization](#file-by-file-customization)).
- **Decide your MSRV policy** and align it across `rust-toolchain.toml`,
  `Cargo.toml` (edition), and CI.
- **Pick a license** and ensure your `Cargo.toml` `license` field matches.

### 2) Clone and start coding

Typical dev loop:

```bash
cargo test --all-features --all-targets
cargo fmt
cargo clippy --all-features --all-targets
```

## What this template gives you

### CI baseline (high level)

The included workflows aim to catch common library pitfalls early:

- **Formatting** (`cargo fmt --check`)
- **Clippy** on **stable** and **beta** (early warning on new lints)
- **Docs build** (runs on nightly using `cargo docs-rs`)
- **Feature combination testing** via `cargo hack --feature-powerset check`
- **MSRV check** (builds on the configured minimal toolchain in CI)
- **Tests** on stable & beta, plus macOS + Windows
- **Minimal dependency versions** (`-Zminimal-versions`) to ensure your version
  requirements are honest
- **Coverage** via `cargo-llvm-cov` + Codecov
- Optional **safety** checks (Miri, sanitizers, Loom)
- Optional **no-std** checks
- Scheduled “rolling” jobs (nightly toolchain + updated dependencies)

### Documentation and project hygiene

You get a baseline set of project meta-files (security policy, code of conduct,
changelog format, issue and PR templates, etc.) so that new crates don’t ship
with “missing basics”.

## MSRV and Rust edition policy

This template is meant for **libraries**, where compatibility matters.

- **MSRV** should be a conscious choice (often ~6–12 months old at project
  creation, unless you need older support).
- Be conservative when bumping MSRV after publishing.
- Use the **minimal Rust edition supported by your MSRV**.

Important: In this repo as-shipped, `rust-toolchain.toml` pins a specific
toolchain, while CI includes an MSRV job. When you instantiate a new project,
you should **make these consistent** for your crate.

## File-by-file customization

This section is intentionally brief: it’s a checklist of what you should update
when you create a new project from this template.

### `Cargo.toml`

- **Must update**:
  - `name`
  - `description`
  - `repository`
  - `license`
  - `authors` (if applicable)
- **Review**:
  - `edition` (must align with your MSRV)
  - `lints.*` (tune warning levels for your team)
  - `profile.release` (keep or adjust based on your needs)
- **Dependencies**:
  - The template intentionally has none. Add only what you need.
  - Prefer specifying **minimally-supported dependency versions** (don’t just
    “float to latest” for libraries).

### `rust-toolchain.toml`

- Decide whether you want:
  - **Pinned toolchain** (reproducibility for contributors), or
  - **Floating stable** (less maintenance)
- If you keep a pinned toolchain, ensure CI still tests stable/beta/nightly as
  appropriate.

### `.github/workflows/*.yml`

- `check.yml`:
  - Formatting, clippy (stable/beta), docs, feature-powerset (`cargo-hack`),
    and MSRV check.
  - **Update the MSRV value** in the matrix to your crate’s MSRV.
- `test.yml`:
  - Tests on stable/beta, minimal dependency versions, OS matrix, coverage.
- `scheduled.yml`:
  - Nightly “rolling” checks and “updated dependencies” checks.
- `safety.yml`:
  - Optional deeper checks (sanitizers, Miri, Loom). Remove if irrelevant.
  - Note: this workflow installs system packages; keep only if you want it.
- `nostd.yml`:
  - Only keep if your crate supports `no_std` (and configure targets/features).

### `.github/dependabot.yml`

- Keeps GitHub Actions up to date.
- For Cargo dependencies, it’s configured with library-friendly defaults.
  Review the ignore rules if your repository also ships binaries.

### `.github/CODEOWNERS`

- Replace `{{code-owner}}` placeholders with the correct GitHub handles/teams.
- Ensure `SECURITY.md` has the right owners.

### `.github/ISSUE_TEMPLATE/*` and `.github/pull_request_template.md`

- Adjust wording to your project.
- Consider adding a PR checklist item that ensures the license is not a
  placeholder (if you use placeholder licenses).

### `rustfmt.toml` and `clippy.toml`

- Tune formatting and clippy settings to team preferences.
- `rustfmt.toml` includes settings that may require nightly rustfmt depending
  on your toolchain; adjust if you need strict stable-only formatting.

### `codecov.yml` and `.github/codecov.yml`

- Decide your coverage target and what to ignore.
- If you use Codecov, configure `CODECOV_TOKEN` in repo secrets as needed.

### `CHANGELOG.md`

- Follows “Keep a Changelog” and Semantic Versioning.
- Keep it up to date for user-visible changes.

### `SECURITY.md`

- Update project name/slug placeholders.
- Ensure the reporting mechanism and links point to your repository.

### `CODE_OF_CONDUCT.md`

- Keep as-is, or replace with your organization’s standard CoC.

### `CONTRIBUTING.md`

- This file is currently a stub in this template repository.
  Fill it with your project’s contribution workflow (local setup, testing
  requirements, release process, etc.).

### `LICENSE-MIT` and `LICENSE-AGPL-3`

- Choose the license(s) that apply to your project.
- Ensure `Cargo.toml` `license = "..."` matches what you ship.
- Remove any licenses you do not intend to ship.

### `src/lib.rs`

- Replace the placeholder code with your crate’s public API.
- Keep `#![deny(missing_docs)]`/docs expectations aligned with your lint
  policy (see `Cargo.toml` lints).

### `.vscode/`

- Optional editor configuration.
- E.g. configures cargo formatter to always run in `nightly` to access latest formatting features.
- Keep, adjust, or remove.

## Notes for existing projects

You can also use this repository as a **reference checklist** for existing Rust
projects:

- Add missing governance/meta-files.
- Adopt the CI workflows incrementally.
- Introduce MSRV + minimal-deps checks to avoid accidental ecosystem breakage.

## Credit

The GitHub Actions CI workflows under `.github/workflows/` are based on the
excellent work in:

- https://github.com/jonhoo/rust-ci-conf/

This template intentionally keeps the same general approach (including minimal
dependency testing and feature-combination checks), because it has proven to be
a solid baseline for Rust libraries.
