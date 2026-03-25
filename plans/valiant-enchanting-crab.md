# Plan: ADHD Todo — to-do.md items

## Context
Working through all open items in `to-do.md`: 2 bugs, 2 feature additions, 1 styling change.

---

## Bug 1 — Drag reorder moves wrong items

**Root cause confirmed:** `useTasks` is called with `includeComplete: true`, so its `tasks` state holds both active and done tasks. `App.tsx` derives `activeTasks` and `doneTasks` via `.filter()`. `TaskList` operates on `activeTasks` only — `handleDragEnd` finds correct indices within that slice — but `reorder` in `useTasks` calls `arrayMove(tasks, oldIndex, newIndex)` on the **full** mixed array. If any done task has a `sort_order` that places it before an active task in the array, indices are off by up to `doneTasks.length`.

**Fix — `src/renderer/src/hooks/useTasks.ts` (`reorder` callback):**
- Split `tasks` into `active` and `done` before `arrayMove`
- Call `arrayMove(active, oldIndex, newIndex)` to get `reorderedActive`
- Call `setTasks([...reorderedActive, ...done])` for optimistic update
- Pass `reorderedActive.map(t => t.id)` to `ipc.reorderTasks` (done tasks excluded — DB already handles them separately)

---

## Bug 2 — Minimize button "closes" the app

**Root cause confirmed:** `minimizeToTray()` correctly calls `mainWindow.hide()`. The tray is created with a right-click context menu ("Open", "Quit") and a `double-click` handler — but **no single-click handler**. Windows users expect a single left-click on a tray icon to restore the window; without it, the app appears to disappear.

**Fix — `src/main/tray.ts` (`createTray`):**
- Add `tray.on('click', ...)` with the same show/focus logic already in the `double-click` handler

---

## Change — DO BEFORE 5PM banner

**Location:** `src/renderer/src/components/banner/Before5pmBanner.tsx`

**Current issues:** aggressive `animate-pulse-border` animation on the non-overdue state, dark `bg-amber-950/80` background, heavy `font-semibold`.

**Fix:**
- Non-overdue state: soften to `bg-amber-900/40 border-amber-400/50 text-amber-300`, use `font-medium`, remove `animate-pulse-border`
- Overdue state: keep stronger styling (`bg-red-900/60 border-red-400/70 text-red-300`) + keep `animate-pulse-border` (urgency is warranted)
- Trigger Overdue State from 4:45pm to 5pm (crunch time)
- Reduce `border-b-2` to `border-b` for both states

---

## Addition — Priority field (1–10)

**Step 1 — Migration** (`src/main/db/migrations.ts`):
- Add try/catch `ALTER TABLE tasks ADD COLUMN priority INTEGER NOT NULL DEFAULT 5` (follow existing `xp_value` migration pattern)

**Step 2 — Shared types** (`src/shared/types.ts`):
- Add `priority: number` to `Task` interface
- Verify `CreateTaskPayload` and `UpdateTaskPayload` Omit lists don't exclude it

**Step 3 — DB layer** (`src/main/db/tasks.ts`):
- `rowToTask`: add `priority: (row.priority as number) ?? 5`
- `createTask`: add `priority` to INSERT, bind `payload.priority ?? 5`
- `updateTask`: add `'priority'` to the update field map
- `completeTask` recurrence path: propagate `priority: task.priority`

**Step 4 — TaskCard** (`src/renderer/src/components/tasks/TaskCard.tsx`):
- Add priority badge in the metadata row — only show when `priority >= 7` (reduces clutter for ADHD users)
  - 8–10: `text-red-300 bg-red-900/50`
  - 7: `text-amber-300 bg-amber-900/50`
  - 1–6: no badge

**Step 5 — TaskForm** (`src/renderer/src/components/tasks/TaskForm.tsx`):
- Add `priority` state, defaulting to `initial?.priority ?? 5`
- Add a labeled `<select>` with grouped options: Low (1–3), Medium (4–6), High (7–10)
- Include `priority` in the save payload

**Step 6 — BrainDumpInput** (`src/renderer/src/components/braindump/BrainDumpInput.tsx`):
- Add `priority: 5` to the hardcoded payload (no UI needed)

---

## Addition — Sort modes

**State** (`src/renderer/src/store/appStore.ts`):
- Add `sortMode: 'free' | 'by-category' | 'by-priority'`
- Add `setSortMode` action, initial value `'free'`

**Sort toggle UI** (`src/renderer/src/components/tasks/TaskList.tsx`):
- Add 3-button segmented control to the header row (alongside "Add task")
- Icons: `List` (free), `Layers` (by-category), `ArrowUpDown` (by-priority) — all from Lucide, with tooltips

**TaskList restructure** (`src/renderer/src/components/tasks/TaskList.tsx`):
- **`free` mode**: current behavior unchanged (single `SortableContext`, current `handleDragEnd`)
- **`by-category` / `by-priority` modes**: compute groups in the component, render one `SortableContext` per group inside one shared `DndContext`
  - Group header between contexts
  - `handleDragEnd`: use `activeTasks.findIndex(t => t.id === active.id/over.id)` directly (avoids sub-index mapping); if dragged task and drop target are in different groups, return early (no cross-group reorder)
- `by-category` groups: tasks grouped by `categoryId`, ordered by category `sortOrder`; null categoryId → "Uncategorized" at end
- `by-priority` groups: High (7–10), Medium (4–6), Low (1–3); priority requires field from Addition above

---

## Implementation order

1. Bug 2 (tray single-click) — `tray.ts` only, zero risk
2. Banner styling — `Before5pmBanner.tsx` only, cosmetic
3. Bug 1 (drag reorder) — `useTasks.ts` only, core fix
4. Priority field — vertical slice across 6 files, no UI dependencies from sort modes
5. Sort modes — depends on priority field (for by-priority grouping)

---

## Verification

| Item | How to test |
|------|-------------|
| Bug 1 | Complete a task, drag-reorder remaining active tasks — they should move correctly |
| Bug 2 | Click minimize button → app hides; single-click tray icon → app restores |
| Banner | Create a task with "Do Before 5 PM" checked → softer banner appears; let it go overdue → urgent red pulsing banner appears |
| Priority | Create task with priority 9 → red badge on card; edit to 3 → badge gone |
| Sort modes | Switch to by-category → tasks grouped by category headers, drag within group reorders, drag across groups has no effect; same for by-priority |
