---
name: ADHD Todo — App Architecture
description: Core architecture, tech stack, and file structure for the adhd-todo Electron desktop app built for a user with ADHD
type: project
---

Desktop to-do app built at `Documents/to-dos/adhd-todo/`. **Project complete as of 2026-03-23.** Packaged exe at `release\win-unpacked\ADHD Todo.exe` (169 MB). Launched via `launch.bat` (double-click).

**Why:** User has ADHD and needs a persistent, ADHD-friendly desktop to-do list.

**How to apply:** Use this context when continuing development on the adhd-todo app.

---

## Tech Stack
- Electron + electron-vite
- React 18 + TypeScript (renderer)
- Tailwind CSS (always dark, `class="dark"` on `<html>`)
- better-sqlite3 (SQLite at `%AppData%/adhd-todo/adhd-todo.db`)
- @dnd-kit (drag-to-reorder)
- date-fns (date math, used in both processes)
- Zustand (ephemeral UI state only)
- lucide-react (icons)

## Key Commands
```bash
cd Documents/to-dos/adhd-todo
npm run dev        # hot reload dev mode
npm run build      # production build
npm run rebuild    # re-run electron-rebuild for better-sqlite3
npm run pack       # build + Windows installer (.exe in release/)
```

## Features Implemented
- System tray (close hides to tray, double-click to reopen)
- Dark mode always-on
- Large live clock (HH:MM:SS) with date
- "DO BEFORE 5PM" amber countdown banner (auto-appears, turns red when overdue)
- Categories sidebar — always fully visible (ADHD: out of sight = forgotten)
- Tasks: title, category, color, due date, time estimate, subtasks, recurring pattern
- Drag-to-reorder with optimistic updates
- Color coding (per-task left border)
- Recurring tasks (daily/weekly/monthly — next occurrence auto-created on complete)
- Brain dump: Ctrl+Space opens quick-add strip; Enter adds, stays open, Escape closes
- Focus mode: full-screen overlay triggered per-task, Escape exits
- Pomodoro timer: 4×(25/5) + 30min long break; SVG ring progress; start/stop
- Pomodoro sessions logged to `pomodoro_sessions` table (ready for gamification)
- GamificationStub component (empty placeholder for Phase 1–3 gamification)
- **Gamification Phase 1**: XP/level bar in sidebar, task completion animation, Pomodoro "today" counter
- **Auto-start on login**: "Start on login" checkbox in system tray menu (`app.setLoginItemSettings`)
- **Launcher**: `launch.bat` (double-click, brief cmd flash) and `launch.vbs` (silent, requires wscript.exe file association)
- **Packaging**: `npm run pack` produces `release\win-unpacked\ADHD Todo.exe`; final winCodeSign symlink error is harmless — exe is complete before it occurs

## IPC Pattern
- Shared types: `src/shared/types.ts`
- Channel constants: `src/shared/ipc-channels.ts` (IPC.TASKS.GET_ALL etc.)
- Main handlers: `src/main/ipc/*.ts`
- Renderer wrappers: `src/renderer/src/lib/ipc.ts` (typed async functions)
- Preload: `src/preload/index.ts` (contextBridge whitelist)

## Next Steps
- Gamification Phase 2–3 (see gamification memory for details)
- Icon upgrade if desired (current is functional 256×256 but simple)
