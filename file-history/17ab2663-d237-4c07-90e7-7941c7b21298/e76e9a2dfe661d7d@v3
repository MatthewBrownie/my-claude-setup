# Context

Audit of `~/.claude` revealed two issues to fix:
1. `claude_rules.md` duplicates `CLAUDE.md` — same content, risk of divergence when one is updated and not the other
2. Memory system directory exists but is uninitiated — no `MEMORY.md` index file

# Task 1: Merge claude_rules.md into CLAUDE.md

`claude_rules.md` is a prior version of the instructions in `CLAUDE.md`. The two files are ~95% identical, but `claude_rules.md` has two additions in the Git section that `CLAUDE.md` is missing:

- `pnpm-lock.yaml` not listed in lockfiles
- "Never commit build artifacts, node_modules, .venv, .pytest_cache, dist, or coverage outputs" — missing from CLAUDE.md entirely

**Plan:**
1. Add `pnpm-lock.yaml` to the lockfiles list in `CLAUDE.md:47`
2. Add a new bullet under Git hygiene: "Never commit build artifacts (`node_modules`, `.venv`, `.pytest_cache`, `dist`, coverage outputs); assume they should be git-ignored."
3. Delete `claude_rules.md`

**Files:**
- Edit: `C:\Users\mbrow\.claude\CLAUDE.md`
- Delete: `C:\Users\mbrow\.claude\claude_rules.md`

# Task 2: Initialize MEMORY.md

Create an empty `MEMORY.md` index at `C:\Users\mbrow\.claude\projects\C--Users-mbrow--claude\memory\MEMORY.md` with a heading and placeholder comment. No memory entries yet — those accumulate over time as the system is used.

**File to create:**
- `C:\Users\mbrow\.claude\projects\C--Users-mbrow--claude\memory\MEMORY.md`

# Verification

- Open `CLAUDE.md` and confirm both git additions are present and the file reads cleanly
- Confirm `claude_rules.md` no longer exists
- Confirm `MEMORY.md` exists at the memory directory path
