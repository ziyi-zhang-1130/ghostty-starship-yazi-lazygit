# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this repo is

A single-file, agent-executable runbook (`INSTALL.md`) for setting up four macOS terminal tools: Ghostty, Starship, Yazi, Lazygit. There is no build, no tests, no source code. The audience declared at the top of `INSTALL.md` is AI agents executing it autonomously — edits must preserve that contract.

## How to "run" it

Execute the shell blocks inside `INSTALL.md` top-to-bottom on macOS. Each section is idempotent. If any `Verify install` or `Verify config` block exits non-zero, halt — do not proceed. The final `End-to-end verification` block at the bottom of `INSTALL.md` is the canonical smoke test; after any edit, re-derive it so it matches the per-section verifications.

## Invariants to preserve when editing

Every tool section has exactly five parts, in order: **Purpose**, **Install**, **Verify install**, **Configure**, **Verify config**. Do not reorder or omit. If a section truly has no configuration, keep the headings and write `None at this stage.` / `N/A.` (see Lazygit for the pattern).

File-write operations use one of three named primitives — do not invent new ones:

- **CREATE** — write only if the file does not exist.
- **OVERWRITE-IF-MANAGED** — write if absent, or if the first line matches the managed marker `# managed-by: ~/install.md`. Otherwise halt with "existing file differs from managed version". The marker must be the literal first line of any file written this way.
- **APPEND-IF-MISSING** — append a block wrapped in `# >>> <name> >>>` / `# <<< <name> <<<` markers; skip if the opening marker is already present in the target file. The name inside the markers is the idempotency key — changing it will cause the block to be re-appended on the next run.

## Editing conventions

- When adding or changing a managed config, update the `Verify config` block in the same commit — the two are a pair.
- The `Upstream source (for humans)` link in the Ghostty section is informational only; the authoritative config is the heredoc in `INSTALL.md`. If you change one, change both.
- `brew install` / `brew install --cask` are treated as idempotent no-ops when already satisfied — rely on that rather than adding `if ! brew list ...` guards.
- The end-to-end verification block duplicates the per-section verifications intentionally. Keep them in sync; treat any divergence as a bug.
- `README.md`'s `customize` section describes the file-op contract to humans — if you change a section's primitive (CREATE / OVERWRITE-IF-MANAGED / APPEND-IF-MISSING), update README in the same commit so the user-facing description doesn't go stale.
