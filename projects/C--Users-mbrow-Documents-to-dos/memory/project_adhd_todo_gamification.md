---
name: ADHD Todo — Gamification Research Findings
description: Research findings on ADHD-effective gamification for the adhd-todo desktop app; includes prioritized feature list, anti-patterns, and SQLite schema additions
type: project
---

Gamification research completed for the adhd-todo Electron app (at `Documents/to-dos/adhd-todo/`).

**Why:** User has ADHD and wants dopamine-supportive gamification that avoids shame spirals.

**How to apply:** Use these findings to implement Phase 1–3 gamification in a follow-up session. Never use punitive mechanics (XP loss, unforgiving streak resets).

---

## Core ADHD Gamification Principles (must not violate these)
1. Never subtract points/XP — only add
2. All rewards must be immediate and visible (not buried in menus)
3. Streaks need grace periods / shields — one miss must never silently reset
4. Celebrate effort, not just outcomes (Pomodoro XP even if task isn't done)
5. Keep it opt-outable — some ADHD users find it distracting

---

## Recommended Features (Priority Order)

### Phase 1 — Foundation (Highest dopamine per effort)
1. **Task completion animation** — CSS particle burst on task complete; zero DB changes needed; pure renderer
2. **XP system + always-visible level bar** — `user_stats` table + `xp_value INT DEFAULT 10` on tasks; level bar in sidebar/header always shown
3. **Pomodoro "today" counter** — `SELECT COUNT(*) FROM pomodoro_sessions WHERE DATE(started_at) = DATE('now') AND was_completed = 1`; already fully logged; combats ADHD time blindness

### Phase 2 — Retention
4. **Forgiving streaks with shields** — `streak_days`, `streak_last_active`, `streak_shields` on `user_stats`; check on app open; 2-day grace before reset; frame as "protect your streak"
5. **Subtask progress bars** — zero DB changes; computed from `subtasks` table; renders on any task with subtasks
6. **"What I Did Today" panel** — zero DB changes; appears on app close or configurable end-of-day; lists completions + pomodoro count; opt-in/dismissable

### Phase 3 — Delight
7. **Badges + variable bonus XP** — `badges` + `user_badges` tables; `xp_multiplier REAL DEFAULT 1.0` on tasks; randomly assign 2.0 to ~10% of new tasks; 10–15 starter badges

---

## Anti-Patterns to Avoid
- Punitive streaks (reset to 0 on first miss) → shame → app abandonment
- XP/points that decrease (Todoist Karma mistake)
- Leaderboards (ADHD users rarely "win" consistency contests)
- Long-horizon-only goals (weeks/months) — must have daily/hourly payoffs
- All-or-nothing task credit — subtask completion must award partial XP
- Hidden progress (profile pages) — must be visible in main layout

---

## Required DB Changes

### New tables:
```sql
CREATE TABLE user_stats (
    id INTEGER PRIMARY KEY,
    total_xp INTEGER DEFAULT 0,
    level INTEGER DEFAULT 1,
    streak_days INTEGER DEFAULT 0,
    streak_last_active DATE,
    streak_shields INTEGER DEFAULT 2,
    created_at DATETIME DEFAULT CURRENT_TIMESTAMP,
    updated_at DATETIME DEFAULT CURRENT_TIMESTAMP
);

CREATE TABLE badges (
    id INTEGER PRIMARY KEY,
    name TEXT NOT NULL,
    description TEXT,
    icon TEXT,           -- emoji or icon identifier
    xp_reward INTEGER DEFAULT 0,
    condition_type TEXT, -- 'tasks_completed' | 'streak_days' | 'pomodoros_today'
    condition_value INTEGER
);

CREATE TABLE user_badges (
    id INTEGER PRIMARY KEY,
    badge_id INTEGER REFERENCES badges(id),
    earned_at DATETIME DEFAULT CURRENT_TIMESTAMP
);
```

### Additions to existing tables:
```sql
ALTER TABLE tasks ADD COLUMN xp_value INTEGER DEFAULT 10;
ALTER TABLE tasks ADD COLUMN xp_multiplier REAL DEFAULT 1.0;
```

(`pomodoro_sessions` already has `was_completed` — no changes needed.)

---

## App Learning Notes (for this user)
- The Habitica RPG model works but can become a hyperfocus trap for ADHD
- Finch's "caring for something else" model bypasses internal motivation deficits — worth considering as a future theme
- Variable ratio rewards (slot machine mechanic) are especially compelling for ADHD dopamine systems
- Today-only counters (not multi-day streaks) are emotionally safer for Phase 1
