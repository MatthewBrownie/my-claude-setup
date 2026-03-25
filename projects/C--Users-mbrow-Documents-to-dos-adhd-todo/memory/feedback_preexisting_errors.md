---
name: Pre-existing TypeScript errors were masked
description: The web typecheck was silently skipped whenever the node typecheck failed
type: feedback
---

The project's `typecheck` script runs `tsc -p tsconfig.node.json && tsc -p tsconfig.web.json`. If the node check fails (exit 1), the web check never runs. Several pre-existing renderer errors (Promise<unknown> casts in ipc.ts, missing payload fields, unused imports) were hidden behind a single node-side error. Once the node errors were fixed, ~25 web errors surfaced all at once.

**Why:** Not a mistake — just a gotcha with chained typecheck scripts.
**How to apply:** When fixing typecheck errors in this project, expect a second wave of renderer errors to appear after the main-process errors are cleared. Don't be surprised — fix them in batch.
