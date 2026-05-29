---
description: Bootstrap a new Rust library crate from the OpenZeppelin/rust-project-template. Use when the user asks to create a new Rust library, initialize a crate from this template, "use the rust-project-template", or instantiate a new project with the template's quality baseline. Runs as a three-phase wizard (metadata → project-type → finalize) with per-step apply, journal-based resume, and `back`/`skip`/`finish` escape hatches.
---

# rust-template-init

Thin Claude Code wrapper. The authoritative, tool-neutral instructions
live in [INIT.md](../../../INIT.md) at the repo root — read and follow
that file.

Claude-Code-specific notes:

- Use `AskUserQuestion` for the pickers in A3, A7, A8, and Phase B.
- Use `Edit` (targeted `old_string`/`new_string`) for all sentinel
  replacements — never `sed`.
- Use `TodoWrite` to track the per-step progress mirrored in
  `.rust-template-init.json`.
