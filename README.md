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

### 1) Automated setup with Claude Code (recommended)

This repo ships with a Claude Code skill at
[.claude/skills/rust-template-init/](.claude/skills/rust-template-init/SKILL.md)
that automates the entire first-time setup checklist below — metadata,
license, MSRV, workflow trimming, starter-content reset, and the initial
commit.

### 2) Manual setup

#### 1) Create a new repository from it (GitHub template)

You can instantiate a new crate by using GitHub’s **“Use this template”** button.

After creation:

- **Rename placeholders** in `Cargo.toml` and the repository docs/templates (see
  [First-time setup checklist](#first-time-setup-checklist) and
  [File-by-file customization](#file-by-file-customization)).
- **Decide your MSRV policy** and align it across `rust-toolchain.toml`,
  `Cargo.toml` (edition), and CI.
- **Pick a license** and ensure your `Cargo.toml` `license` field matches.

#### 2) Clone and start coding

**Quickstart:**

```bash
git clone --depth 1 https://github.com/OpenZeppelin/rust-project-template.git my-crate
cd my-crate
rm -rf .git && git init -b main
claude  # then prompt: "init a new rust crate from this template"
```

The skill runs as a three-phase wizard:

1. **Metadata** — one question at a time, with smart defaults and
   suggestions (crate name, owner, repo URL, description, authors, MSRV,
   license, keywords, categories, CODEOWNERS, security contact).
   Each answer is applied and committed immediately, so you can
   `back` / `skip` / `finish` / `abort` at any point.
2. **Project type** — batched prompt for `std` vs `no_std`, safety CI,
   crates.io publishing, Codecov, benches, examples. Preview shown
   before destructive changes.
3. **Finalize** — resets `src/lib.rs`, `CHANGELOG.md`, and wipes the
   README down to a single `# <crate-name>` heading. Runs
   fmt/clippy/test. Offers the initial commit.

A `.rust-template-init.json` journal in the repo root lets you resume an
interrupted run. It is deleted on successful completion.

If you prefer to do it by hand, the [First-time setup checklist](#first-time-setup-checklist)
below is the same list of changes in manual form.

## First-time setup checklist

If this is your first time creating a repository from this template, use this
checklist before you start writing code:

1. **Update crate/package metadata in `Cargo.toml`**
   - Replace `name` with your crate name.
   - Replace `description` with a real one-line summary.
   - Replace `repository` with your repository URL.
   - Replace `authors` with the actual maintainers, if you use that field.
   - Replace `license` if you are not shipping MIT.
2. **Choose which license files to keep**
   - Keep only the license file(s) that apply to your project.
   - Make sure the `Cargo.toml` `license` field matches the files you ship.
3. **Replace organization/project-specific names and links**
   - Search the repository for the template values and replace them with your
     own project details.
   - At minimum, search for:
     - `rust-project-template`
     - `OpenZeppelin`
     - `openzeppelin`
     - `0xNeshi`
     - `project description`
4. **Update maintainer/contact information**
   - Set the correct owners in `.github/CODEOWNERS`.
   - Update security reporting instructions in `SECURITY.md`.
   - Update support/community links in GitHub issue templates if you use them.
5. **Review CI and toolchain policy**
   - Set your MSRV in workflow files.
   - Confirm `edition` is compatible with your MSRV.
   - Decide whether `rust-toolchain.toml` should stay pinned.
6. **Trim anything you do not use**
   - Remove workflows you do not need (`nostd.yml`, `safety.yml`, etc.).
   - Remove example/bench scaffolding if it is not useful for your crate.

After the checklist, run a sanity check:

```bash
cargo test --all-features --all-targets
cargo +nightly fmt --check
cargo clippy --all-features --all-targets
```

## What this template gives you

### CI baseline (high level)

The included workflows aim to catch common library pitfalls early:

- **Formatting** (`cargo fmt --check`)
- **Spell-check** of source and docs (via [`typos`](https://github.com/crate-ci/typos))
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

This section is a concrete map of which files usually need project-specific
names, links, maintainers, and policy choices changed after you create a new
repository from this template.

### `Cargo.toml`

- **Must update**:
  - `name`
  - `description`
  - `repository`
  - `license`
  - `authors` (if applicable)
  - `keywords` and `categories` once you know how you want to publish the crate
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
  - Review repository-specific job names, notifications, and assumptions.
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

- Add correct GitHub handles/teams.
- Ensure `SECURITY.md` has the right owners.

### `.github/ISSUE_TEMPLATE/*` and `.github/pull_request_template.md`

- Adjust wording to your project.
- Replace support links, repository links, and organization names.

### `rustfmt.toml` and `clippy.toml`

- Tune formatting and clippy settings to team preferences.
- `rustfmt.toml` includes settings that may require nightly rustfmt depending
  on your toolchain; adjust if you need strict stable-only formatting.
- `clippy.toml` may include organization-specific identifiers; review
  `doc-valid-idents`.

### `codecov.yml` and `.github/codecov.yml`

- Decide your coverage target and what to ignore.
- If you use Codecov, configure `CODECOV_TOKEN` in repo secrets as needed.

### `CHANGELOG.md`

- Follows “Keep a Changelog” and Semantic Versioning.
- Keep it up to date for user-visible changes.

### `SECURITY.md`

- Update the project name, organization name, and repository slug.
- Replace the vulnerability reporting contact/channel with your own.
- Ensure the advisory submission link points to your repository.
- Review any legal text inherited from the template owner.

### `CODE_OF_CONDUCT.md`

- Keep as-is, or replace with your organization’s standard CoC.
- If you keep this file, review the enforcement/reporting contact details.

### `CONTRIBUTING.md`

- This file is currently a stub in this template repository.
  Fill it with your project’s contribution workflow (local setup, testing
  requirements, release process, etc.).

### `LICENSE-MIT` and `LICENSE-AGPL-3`

- Choose the license(s) that apply to your project.
- Ensure `Cargo.toml` `license = "..."` matches what you ship.
- Remove any licenses you do not intend to ship.
- If your chosen license requires copyright holder updates, make those changes.

### `src/lib.rs`

- Replace the placeholder code with your crate’s public API.
- Keep `#![deny(missing_docs)]`/docs expectations aligned with your lint
  policy (see `Cargo.toml` lints).

### `.vscode/`

- Optional editor configuration.
- E.g. configures cargo formatter to always run in `nightly` to access latest formatting features.
- Keep, adjust, or remove.

## Recommended repository-wide search

Before your first release, do one repository-wide search for template values and
verify each remaining match is intentional.

Suggested search terms:

- `rust-project-template`
- `OpenZeppelin`
- `openzeppelin`
- `0xNeshi`
- `project description`

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
