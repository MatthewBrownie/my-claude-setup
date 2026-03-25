# Plan: Write README.md for ~/.claude repo

## Context
The ~/.claude directory is being set up as a private git repo synced between desktop and laptop.
A README is needed to document what the repo is, what's tracked vs ignored, how to set up on a new machine, and a skills reference.

## Output
**File:** `C:\Users\mbrow\.claude\README.md`

## Content sections
1. **What this is** — one-liner: global Claude Code config synced between machines
2. **What's tracked** — CLAUDE.md, settings.json, agents/, skills/, james/, plugins/, projects/memory/
3. **What's gitignored** — .credentials.json (handle separately), history.jsonl, cache/, sessions/, shell-snapshots/
4. **New machine setup** — clone, place at ~/.claude, copy .credentials.json manually
5. **Skills** — table listing the three skills and their purpose

## Verification
Confirm file exists at `C:\Users\mbrow\.claude\README.md` and reads cleanly.
