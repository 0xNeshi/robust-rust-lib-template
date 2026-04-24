---
description: Bootstrap a new Rust library crate from the OpenZeppelin/rust-project-template. Use when the user asks to create a new Rust library, initialize a crate from this template, "use the rust-project-template", or instantiate a new project with the template's quality baseline. Runs as a three-phase wizard (metadata → project-type → finalize) with per-step apply, journal-based resume, and `back`/`skip`/`finish` escape hatches.
---

# rust-template-init

Automate the "First-time setup checklist" from the
[OpenZeppelin/rust-project-template](https://github.com/OpenZeppelin/rust-project-template)
README so a developer ends up with a clean, ready-to-code library crate.

## When to use

Trigger on any of:

- "create a new rust library using the openzeppelin template"
- "bootstrap a crate from rust-project-template"
- "init a new rust project from this template"
- user provides the template URL and asks you to instantiate it
- a `.rust-template-init.json` journal exists in CWD → **resume mode**

## Core principles

- **Ask one thing at a time in Phase A.** Smart defaults build on prior
  answers (repo URL suggests `<owner>/<name>` only after owner + name
  are known).
- **Batch related destructive decisions in Phase B and C**, show a
  preview diff, then apply atomically after one confirmation.
- **Never fabricate.** If the user skips a field that can't be inferred,
  leave a `TODO(rust-template-init): <what>` marker and list it in the
  final summary.
- **Commit after every applied step** so abandoning mid-flow leaves a
  clean, per-decision git history.
- **Validate inline** — do not defer errors to the end.

## Universal commands (available at every prompt)

- `skip` — skip this step, leave value at template default, continue.
- `back` — revert the last applied step (git reset + pop journal) and
  re-ask.
- `finish` — stop asking; apply remaining phases with defaults and
  proceed straight to Phase C.
- `abort` — exit without further changes; journal file is preserved so
  the user can `resume` later.

Announce these commands at the start of Phase A.

## Journal (state file)

Write progress to `.rust-template-init.json` at the repo root (add to
`.gitignore` if not already). Schema:

```json
{
  "phase": "A|B|C|done",
  "step": "<current-step-id>",
  "answers": { "name": "...", "owner": "...", "...": "..." },
  "applied_commits": ["<sha>", "..."],
  "todos": [{ "marker": "TODO(rust-template-init): ...", "file": "...", "line": 0 }]
}
```

If the file exists when the skill starts, ask: *"Found an in-progress
setup. Resume from step `<step>`, start over, or abort?"*

Delete the journal on successful completion.

## Phase A — Metadata wizard

One step, one question, one apply, one commit. Order matters: later
suggestions use earlier answers.

For each step: show `[i/N] <step name>`, print the question, show the
default in brackets, accept the answer, validate, apply, commit with
message `rust-template-init: <step-id>`.

### A1. Crate name `[default: <CWD basename>]`

- **Validate**: ASCII alphanumeric + `_`/`-`, ≤64 chars, non-empty.
- **Warn** (don't block) if `cargo search <name>` shows an exact match.
- **Apply**: replace token `rust-project-template` repo-wide with
  `<name>`; update `Cargo.toml` `name`.

### A2. GitHub owner slug `[default: gh api user -q .login || git config user.name]`

- **Validate**: GitHub username regex (`^[a-z\d](?:[a-z\d]|-(?=[a-z\d])){0,38}$i`).
- **Apply**: replace `openzeppelin` (case-insensitive, word-boundary)
  with the slug; replace `OpenZeppelin` (display form) with the
  user-supplied display name (re-ask if the slug and display differ).

### A3. Repository URL — offer a picker

```
  [default] https://github.com/<owner>/<name>
  [alt]     https://github.com/<gh-username>/<name>
  [custom]  (type your own)
  [skip]
```

Use `AskUserQuestion` for the picker. **Apply**:

- `Cargo.toml` `repository`, issue templates, SECURITY, CONTRIBUTING links.
- **Configure git remote** so the local repo knows where to push:
  - If an `origin` remote does **not** exist: run
    `git remote add origin <url>`.
  - If `origin` exists and points elsewhere: show the current URL and
    ask whether to overwrite. On yes: `git remote set-url origin <url>`.
    On no: leave it untouched and record a
    `TODO(rust-template-init): set git remote origin to <url>` in the
    journal.
  - For SSH-preferring users, after setting the HTTPS form, ask whether
    to convert to SSH (`git@github.com:<owner>/<name>.git`). Default: no.
- **Do not push.** Creating the remote on GitHub and pushing is left to
  the user. Hint in the final summary:
  `git push -u origin main` (only after you create the repo on GitHub).
- **Do not run `gh repo create`** automatically — creating a remote
  repository is a user decision that may conflict with org policy,
  visibility defaults, or naming. Offer it only if the user explicitly
  asks.

### A4. One-line description

- **Validate**: 10–140 chars, no trailing period (crates.io convention).
- **Apply**: replace `project description` token repo-wide;
  `Cargo.toml` `description`.

### A5. Authors `[default: "<git user.name> <git user.email>"]`

- Accept comma-separated list.
- **Apply**: `Cargo.toml` `authors`.

### A6. Primary maintainer GitHub handle `[default: owner slug]`

- **Apply**: replace `0xNeshi` with the handle.

### A7. MSRV — offer a picker

```
  [default] 1.85.0  (template's current value, matches edition 2024)
  [recent]  <stable - 3 releases>  (e.g. 1.82.0 if current stable is 1.85)
  [custom]  (type your own, must be x.y.z)
  [skip]
```

- **Validate**: parses as semver, ≤ current stable.
- **Apply**: `Cargo.toml` `rust-version`; `check.yml` `msrv` matrix;
  `rust-toolchain.toml` `channel` (ask: keep pinned, float stable, or
  delete the file).

### A8. License — offer a picker

```
  [default] MIT
  [alt]     AGPL-3.0-only
  [dual]    MIT OR AGPL-3.0-only
  [custom]  (type any SPDX expression)
  [skip]
```

**Apply** (destructive — confirm the file deletions in one batch):

- `MIT` → delete `LICENSE-AGPL-3`, rename `LICENSE-MIT` → `LICENSE`.
- `AGPL-3.0-only` → delete `LICENSE-MIT`, rename `LICENSE-AGPL-3` → `LICENSE`.
- `MIT OR AGPL-3.0-only` → keep both files **as-is** (do not rename —
  the `LICENSE-<NAME>` convention is required for dual-license clarity).
- `custom <SPDX>` → **validate** against the SPDX license list. Delete
  both template license files, create an empty `LICENSE` file, and add
  `TODO(rust-template-init): paste license text into LICENSE` to the
  journal.

In **all** cases also update `Cargo.toml` `license = "<SPDX>"` to match.

### A9. Keywords (≤5) — optional `[default: skip]`

- Suggest up to 5 by extracting nouns from the description and
  cross-checking against a small curated list of common Rust crate
  keywords.
- **Apply**: `Cargo.toml` `keywords`.

### A10. Categories (≤5) — optional `[default: skip]`

- Suggest from the **fixed** crates.io category list
  (https://crates.io/category_slugs). Do not invent categories — only
  offer slugs that exist. If the curated list is unavailable, skip with
  a TODO.
- **Apply**: `Cargo.toml` `categories`.

### A11. CODEOWNERS `[default: @<primary-handle>]`

- Accept space-separated handles/teams.
- **Apply**: `.github/CODEOWNERS`.

### A12. Security reporting contact

- Accept an email **or** an advisory URL (e.g.
  `https://github.com/<owner>/<name>/security/advisories/new`).
- **Validate**: email regex or URL parses.
- **Apply**: `SECURITY.md`.

At the end of Phase A, commit: `rust-template-init: complete metadata`.

## Phase B — Project type (batched)

Ask all of these **together** via `AskUserQuestion`, show a preview of
the resulting file-system changes, confirm once, apply atomically.

- **Runtime target**
  - `std` (default) → delete `.github/workflows/nostd.yml`.
  - `no_std` → keep it; ask for target triples (e.g.
    `thumbv7em-none-eabi`); add `#![cfg_attr(not(test), no_std)]` to
    `src/lib.rs` in Phase C.
- **Deep-safety CI (Miri + sanitizers + Loom)?**
  - Yes (default for unsafe/concurrency crates) → keep.
  - No → delete `.github/workflows/safety.yml`.
- **Publish to crates.io?**
  - Yes → remove the `if: ${{ false }}` gate on the `semver` job in
    `check.yml`.
  - No → leave gated.
- **Codecov coverage uploads?**
  - Yes → keep.
  - No → delete the `coverage` job in `test.yml`, `codecov.yml`, and
    `.github/codecov.yml`.
- **Keep `benches/` scaffolding?** Delete dir + `Cargo.toml` `[[bench]]`
  entries if no.
- **Keep `examples/` scaffolding?** Delete dir if no.

Preview format:

```
Will apply the following:
  delete  .github/workflows/nostd.yml       (std-only crate)
  delete  .github/workflows/safety.yml      (no unsafe/concurrency)
  keep    semver gate                       (not publishing)
  delete  coverage job, codecov.yml         (Codecov declined)
  delete  benches/                          (not needed)
  keep    examples/
Proceed? [y/N]
```

Commit: `rust-template-init: configure project type`.

## Phase C — Finalize (batched)

Present as one preview, confirm, apply:

- **Reset `src/lib.rs`** to:
  ```rust
  //! <description>
  ```
  Prepend `#![cfg_attr(not(test), no_std)]` if `no_std` was chosen.
- **Reset `CHANGELOG.md`** to a fresh "Keep a Changelog" / SemVer
  header with an empty `[Unreleased]` section.
- **Wipe `README.md`** down to only:
  ```markdown
  # <name>
  ```
  (All template-usage documentation is discarded — the README now
  belongs to the new crate.)
- **Remove `.claude/skills/rust-template-init/`** *unless* the user
  opts to keep it. Default: remove (it was single-use).
- **Run sanity checks**:
  ```bash
  cargo +nightly fmt --all --check
  cargo clippy --all-features --all-targets -- -D warnings
  cargo test --all-features --all-targets
  ```
  For `no_std`: also `cargo check --no-default-features --target <T>`.
- **Offer initial commit**: `chore: initial commit from rust-project-template`.
- **Delete `.rust-template-init.json`**.
- **Print the push hint** if `origin` was configured in A3:
  `git push -u origin main  # after creating the repo on GitHub`.

## Inline validation details

- **Crate name**: regex `^[a-zA-Z][a-zA-Z0-9_-]{0,63}$`. Warn on
  duplicate via `cargo search`.
- **MSRV**: `x.y.z` semver; not greater than `rustc -V` on the
  developer's machine (if `rustc` is available).
- **SPDX license**: match against the SPDX license list. If not found,
  reject and offer to fall back to dual `MIT OR AGPL-3.0-only`.
- **URLs**: parse as URL; optional HEAD with 3 s timeout.
- **Email**: basic RFC5322-lite regex.

## Rules of engagement

- Never skip the preview before a destructive batch.
- **Configure** the local `origin` remote when the user provides a
  repository URL (A3), but **never push** and **never run
  `gh repo create`** — creating the remote repository is the user's
  decision. Confirm before overwriting an existing `origin` URL.
- Never fabricate handles, emails, URLs, or license text.
- Always add new `TODO(rust-template-init): …` markers to the journal
  so the final summary lists them.
- Prefer `Edit` with targeted `old_string`/`new_string` over `sed` so
  no replacement silently over-matches.

## Final summary format

```
✓ rust-template-init complete

Crate:       <name> v0.1.0 (<license>)
Owner:       <owner>
Repo:        <repo-url>
MSRV:        <msrv>
Type:        <std|no_std>

Kept:
  .github/workflows/{check,test,scheduled,safety,nostd}.yml
  benches/, examples/
Removed:
  LICENSE-AGPL-3
  .github/workflows/nostd.yml
  ...
Outstanding TODOs (<N>):
  TODO(rust-template-init): paste license text into LICENSE
  ...
Sanity:
  fmt     ✓
  clippy  ✓
  test    ✓

Next steps:
  Create the GitHub repo at <repo-url>, then:
    git push -u origin main
```
