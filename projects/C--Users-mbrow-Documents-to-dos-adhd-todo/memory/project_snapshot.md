---
name: Project snapshot
description: Stack, DB location, key quirks, and features as of 2026-03-24
type: project
---

**Stack:** Electron + React + TypeScript + SQLite (better-sqlite3) + Tailwind. IPC via contextBridge whitelist in preload.

**DB location:** `%APPDATA%\adhd-todo\adhd-todo.db`

**launch.bat quirk:** Checks for `release\win-unpacked\ADHD Todo.exe` first — if found, runs the packaged exe (which won't have new changes). Delete that folder or run `npm run pack` to get changes reflected.

**Features shipped as of 2026-03-24:**
- Categories, recurring tasks
- Task + subtask descriptions
- Done section (completed tasks persist, collapsible)
- Collapsible sidebar (full panel hide/show)
- Focus mode with subtask management and "mark complete" + auto-advance
- Pomodoro timer (4×25/5 + 30 long break)
- Brain dump (Ctrl+Space)
- XP/level gamification
- Work timer (clock in/out, persisted to `work_sessions` table — historical data ready for future display)
- System tray (single-click to restore)
- **Priority field** (1–10, `priority` column in tasks table, default 5; badge shown for P7+)
- **Sort modes** — free / by-category / by-priority; segmented control in TaskList header; grouped SortableContext per group, cross-group drags silently cancelled
- **Drag-to-reorder** — fixed: `useTasks.reorder` now splits active/done before `arrayMove` so done tasks don't corrupt indices
- **DO BEFORE 5PM banner** — softened; `isCrunchTime` (< 15 min remaining) triggers urgent red pulse from 4:45 PM onward

**Why:** `work_sessions` table was designed to support a future "time worked per day" history view — schema is already in place.

**Drag bug root cause (for future reference):** `useTasks` is called with `includeComplete: true` so its state holds active + done tasks. `App.tsx` derives `activeTasks` via `.filter()` and passes that to `TaskList`. Indices from `handleDragEnd` are relative to `activeTasks`, but old code called `arrayMove` on the full mixed array — off by up to `doneTasks.length`.
