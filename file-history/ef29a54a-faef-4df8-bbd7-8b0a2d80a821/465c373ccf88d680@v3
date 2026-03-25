# ADHD Todo — Phase 2: Gamification + Launcher + Auto-start

## Context
Three additions to the existing Electron app at `Documents/to-dos/adhd-todo/`:
1. **Gamification Phase 1** — task completion animation, XP/level bar in sidebar, Pomodoro "today" counter
2. **Auto-start on login** — toggle in system tray menu using `app.setLoginItemSettings()`
3. **Clickable launcher** — `launch.vbs` script so the user can double-click to open the app without touching a terminal

---

## 1. Gamification Phase 1

### DB changes (`src/main/db/migrations.ts`)
Add to `runMigrations()` — safe to re-run (uses `IF NOT EXISTS` / `IF NOT COLUMN`):
```sql
CREATE TABLE IF NOT EXISTS user_stats (
  id         INTEGER PRIMARY KEY,
  total_xp   INTEGER NOT NULL DEFAULT 0,
  level      INTEGER NOT NULL DEFAULT 1,
  updated_at TEXT    NOT NULL DEFAULT (datetime('now'))
);
INSERT OR IGNORE INTO user_stats (id, total_xp, level) VALUES (1, 0, 1);
ALTER TABLE tasks ADD COLUMN xp_value INTEGER NOT NULL DEFAULT 10;
```
> SQLite doesn't support `ALTER TABLE ADD COLUMN IF NOT EXISTS`. Wrap in a try/catch in JS so re-runs don't crash.

### New file: `src/main/db/gamification.ts`
- `getStats(db)` → `UserStats` — fetches row id=1 from `user_stats`
- `addXp(db, amount)` → `UserStats` — increments `total_xp`, recalculates `level = floor(total_xp / 100) + 1`, returns updated stats
- `getTodayPomodoroCount(db)` → `number` — `SELECT COUNT(*) WHERE DATE(started_at) = DATE('now') AND was_completed = 1 AND phase = 'work'`

### IPC (`src/shared/ipc-channels.ts`)
Add:
```ts
GAMIFICATION: {
  GET_STATS: 'gamification:get-stats',  // returns UserStats + todayPomodoros
}
```

### IPC handler (`src/main/ipc/gamification.ts` + register in `ipc/index.ts`)
- `gamification:get-stats` → `{ ...getStats(db), todayPomodoros: getTodayPomodoroCount(db) }`

### Hook XP into task completion (`src/main/ipc/tasks.ts`)
In the `IPC.TASKS.COMPLETE` handler: after calling `db.completeTask()`, call `gamification.addXp(db, task.xp_value)` and return `{ task, xpGained, stats }` instead of just `task`.
> Update `completeTask` in `db/tasks.ts` to also read `xp_value` from the row.

### Shared types (`src/shared/types.ts`)
Add:
```ts
export interface UserStats {
  totalXp: number
  level: number
  updatedAt: string
}
export interface CompleteTaskResult {
  task: Task
  xpGained: number
  stats: UserStats
  todayPomodoros: number
}
```
Add `xpValue: number` to `Task` interface.

### New hook: `src/renderer/src/hooks/useUserStats.ts`
- State: `{ totalXp, level, todayPomodoros, xpGained }`
- `refresh()` → calls `gamification:get-stats`
- `onTaskComplete(result: CompleteTaskResult)` → sets `xpGained` for animation, merges new stats, calls `refresh()` to update pomodoro count
- `xpGained` clears after 2s (used to show floating "+10 XP" label)

### New component: `src/renderer/src/components/gamification/XpBar.tsx`
Inserted in `Sidebar.tsx` between the header `<div>` and the `<nav>`:
```
[Level 3]  ████████░░░░  80/100 XP
🍅 4 today
```
- Level badge: `Lv. N` in primary color
- XP bar: Tailwind `bg-primary` fill, width = `(totalXp % 100)%`, `transition-all duration-700`
- Floating `+XP` toast: absolute-positioned, animates upward and fades (CSS keyframe), shown when `xpGained > 0`
- Pomodoro counter: tomato emoji + "N today"

### Completion animation (`src/renderer/src/components/tasks/TaskCard.tsx`)
- Add `completing` state (boolean)
- On checkbox click: set `completing = true` → wait 600ms → call `onComplete()`
- While `completing`: card gets `scale-95 opacity-50 bg-primary/10` transition classes
- Check icon briefly shows a filled `CheckSquare` in primary color before card disappears
- Add `animate-complete` keyframe to `globals.css`: flash green, scale up to 1.05, then scale down to 0.95 + fade

### Wire to App.tsx
- `useUserStats` called in `App.tsx`
- `handleCompleteTask` updated: receives `CompleteTaskResult`, passes `xpGained` / `stats` to `useUserStats.onTaskComplete()`
- `XpBar` receives stats as props from `App` → passed down to `Sidebar`

---

## 2. Auto-start on login (`src/main/tray.ts`)

Replace hardcoded `contextMenu` with a function `buildContextMenu(mainWindow)` that reads current state:
```ts
function buildContextMenu(mainWindow: BrowserWindow, tray: Tray): Menu {
  const openAtLogin = app.getLoginItemSettings().openAtLogin
  return Menu.buildFromTemplate([
    { label: 'Open', click: () => { mainWindow.show(); mainWindow.focus() } },
    { type: 'separator' },
    {
      label: 'Start on login',
      type: 'checkbox',
      checked: openAtLogin,
      click: () => {
        app.setLoginItemSettings({ openAtLogin: !openAtLogin })
        tray.setContextMenu(buildContextMenu(mainWindow, tray))
      }
    },
    { type: 'separator' },
    { label: 'Quit', click: () => app.quit() }
  ])
}
```
Call `tray.setContextMenu(buildContextMenu(mainWindow, tray))` on creation.

> Note: `setLoginItemSettings` works correctly in packaged builds. In `npm run dev` it registers `node.exe` — it won't do useful auto-start until the app is packaged with `npm run pack`.

---

## 3. Clickable Launcher (`launch.vbs` in project root)

A VBScript that opens the app silently (no black cmd window):
```vbs
Set WshShell = WScript.CreateObject("WScript.Shell")
WshShell.Run "cmd /c cd /d ""C:\Users\mbrow\Documents\to-dos\adhd-todo"" && npm run dev", 0, False
Set WshShell = Nothing
```
- `0` = hidden window mode
- `False` = don't wait for process to finish (async launch)
- User can right-click → "Create shortcut", then drag shortcut to desktop or pin to taskbar

---

## Files Changed Summary
| File | Change |
|---|---|
| `src/main/db/migrations.ts` | Add `user_stats` table + `xp_value` column |
| `src/main/db/gamification.ts` | **NEW** — `getStats`, `addXp`, `getTodayPomodoroCount` |
| `src/main/db/tasks.ts` | `completeTask` returns `{ task, xpGained }` |
| `src/main/ipc/gamification.ts` | **NEW** — IPC handler |
| `src/main/ipc/tasks.ts` | Call `addXp` after complete; return richer result |
| `src/main/ipc/index.ts` | Register gamification handler |
| `src/main/tray.ts` | Add "Start on login" checkbox toggle |
| `src/shared/types.ts` | Add `UserStats`, `CompleteTaskResult`, `xpValue` on `Task` |
| `src/shared/ipc-channels.ts` | Add `GAMIFICATION` channels |
| `src/renderer/src/lib/ipc.ts` | Add `getStats()`, update `completeTask()` return type |
| `src/renderer/src/hooks/useUserStats.ts` | **NEW** — stats + xpGained state |
| `src/renderer/src/components/gamification/XpBar.tsx` | **NEW** — level, XP bar, pomodoro counter |
| `src/renderer/src/components/tasks/TaskCard.tsx` | Add `completing` state + animation |
| `src/renderer/src/components/layout/Sidebar.tsx` | Add `XpBar` between header and nav |
| `src/renderer/src/globals.css` | Add `@keyframes task-complete` |
| `src/renderer/src/App.tsx` | Wire `useUserStats`, pass stats to Sidebar |
| `launch.vbs` | **NEW** — silent double-click launcher |

---

## Verification
1. `npm run dev` → app launches
2. Complete a task → card flashes, then fades out; XP bar in sidebar animates forward; "+10 XP" floats up
3. Complete a Pomodoro session → "N today" counter in sidebar increments
4. Tray right-click → "Start on login" checkbox toggles; run `npm run pack`, install, toggle → verify auto-start in Task Manager > Startup
5. Double-click `launch.vbs` → app opens, no cmd window visible
