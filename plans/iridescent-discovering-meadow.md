# Plan: Timer Persistence Fix + Pomodoro Task Completion

## Context
Two issues:
1. `WorkTimer` unmounts when the sidebar collapses (it's conditionally rendered), killing the hook state and stopping the clock.
2. Focus mode only shows the task title — user wants to complete the task and tick off subtasks without leaving focus mode, with auto-advance to the next task.

---

## Fix 1 — Timer survives sidebar collapse

**Root cause:** `useWorkTimer()` lives inside `<WorkTimer />`, which is only rendered when `!sidebarCollapsed`. Collapsing = unmount = hook destroyed.

**Fix:** Lift `useWorkTimer()` to `App.tsx`. Pass the four values (`isRunning`, `sessionSeconds`, `todayTotalSeconds`, `toggle`) down as props through `Sidebar` → `WorkTimer`.

- `src/renderer/src/hooks/useWorkTimer.ts` — no change
- `src/renderer/src/components/clock/WorkTimer.tsx` — convert from calling the hook itself to accepting props: `{ isRunning, sessionSeconds, todayTotalSeconds, toggle }`
- `src/renderer/src/components/layout/Sidebar.tsx` — add four props to `SidebarProps`; pass them through to `<WorkTimer />`
- `src/renderer/src/App.tsx` — call `useWorkTimer()`, pass values to `<Sidebar />`

---

## Fix 2 — Task + subtask management inside Focus Mode

**What to add to `FocusMode.tsx`:**
- Task description (if present), shown below the title
- `<SubtaskList taskId={task.id} />` (reuse existing component — already handles toggle/add/delete/description)
- A "Mark Complete" button — calls `onComplete(task.id)`, then auto-advances to the next task

**Auto-advance logic:** `activeTasks` is passed to `FocusMode`. After completing, the current task is removed from that list. The next task is `activeTasks[0]` if it exists (since tasks are already sorted by `sort_order`). If none remain, call `exitFocusMode()`.

**Files:**
- `src/renderer/src/components/focus/FocusMode.tsx` — add `onComplete` prop, description display, `<SubtaskList />`, "Mark Complete" button, auto-advance
- `src/renderer/src/App.tsx` — pass `handleCompleteTask` and `activeTasks` to `<FocusMode />`

Note: `activeTasks` is already computed in App.tsx (`tasks.filter(t => !t.isComplete)`). `SubtaskList` is already fully functional and needs no changes.

---

## Verification
1. Clock in → collapse sidebar → re-expand → timer still running and elapsed time is correct
2. Enter focus mode on a task → description and subtasks visible
3. Tick a subtask checkbox → it marks complete inline
4. Click "Mark Complete" on the task → task moves to Done section, focus mode auto-advances to the next task (or exits if no tasks remain)
