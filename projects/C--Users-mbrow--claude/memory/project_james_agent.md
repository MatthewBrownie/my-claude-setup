---
name: James agent system
description: Summary of the Jamesian agent system built to help the user prepare for meetings with their software architect James
type: project
---

Built a three-agent system emulating the user's software architect James (British, Socratic, efficiency-obsessed, mentorship-focused).

**Files created:**
- `~/.claude/agents/james.md` — opus-powered manager; holds full persona, orchestrates sub-agents, reads notes on startup, updates notes at session end
- `~/.claude/agents/james-reviewer.md` — sonnet sub-agent; reads code and returns structured findings for James to deliver in character
- `~/.claude/agents/james-prep.md` — sonnet sub-agent; compiles meeting prep docs and updates agent notes
- `~/.claude/james/my-notes.md` — user-written notes (meeting notes, dev observations); James reads, never writes
- `~/.claude/james/agent-notes.md` — agent-managed persistent memory; structured as Standing concerns / Active topics / Session log / Learning journey

**Key design decisions:**
- Persona lives entirely in the manager; sub-agents are personality-free workers
- Two-note memory system: user owns one, agent owns the other
- Pruning rules: completed/old topics condense to one line but are never deleted; session log entries expire after 30 days; standing concerns never pruned
- Learning journey tracks concept mastery (introduced → practicing → understood); James never re-explains understood concepts, asks questions before explaining new ones

**Why:** User has recurring meetings with the real James and wants to stress-test architecture, review code, and prepare questions beforehand in James's authentic style.

**How to apply:** When the user mentions James, architecture prep, or asks to spin up the James agent, reference this system. The agent is invoked with "Use the james agent to...".
