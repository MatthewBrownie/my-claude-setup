# Plan: `function-diagram` agent

## Context

The user wants a new Claude Code agent that takes the structured Markdown output
from the existing `function-tracer` agent and generates a visual call graph
image using the nano-banana skill (Gemini CLI image generation).

---

## What to create

**File**: `~/.claude/agents/function-diagram.md`

---

## Agent design

### Frontmatter

```yaml
name: function-diagram
description: >
  Accepts a function-tracer report and generates a call graph diagram image.
  Extracts callers, callees, and data flow touchpoints from the report, then
  calls the Gemini CLI to render a directed call graph. Use after function-tracer
  when you want a visual of a function's dependencies.
tools: Bash(gemini:*)
model: sonnet
```

### Behavior (numbered steps in system prompt)

1. **Parse the report** — extract from each section:
   - `## Definition` → function name (file-qualified if ambiguous)
   - `## Call Sites` → enclosing scope per entry (deduplicated) = callers
   - `## Internal Calls` → callee names (strip file/line metadata)
   - `## Data Flow` → group by type (DB, file I/O, HTTP, queue, shared state) into labeled nodes

2. **Construct a `/diagram` prompt** — a single plain-English paragraph (no bullet points, no newlines — must be a valid quoted shell argument) that:
   - Names the central node (the traced function + file)
   - Lists callers with arrows pointing INTO the function
   - Lists callees with arrows pointing OUT of the function
   - Adds dashed-edge side nodes for data flow (cylinder=DB, cloud=network)
   - Ends with explicit style directive: "directed call graph, not a flowchart; left-to-right layout, callers left, central function center, callees right"

3. **Show prompt before running** — print under `### Diagram prompt` for user review (cheap, avoids wasted API calls)

4. **Run** `gemini --yolo "/diagram '[PROMPT]' --type='call-graph' --style='technical'"`

5. **Report output** — `ls -t ./nanobanana-output/` to find latest file, present path, offer to regenerate

### Constraints in system prompt

- Operate only on the report text provided — no file reads
- If `## Definition` or `## Call Sites` is missing, stop and tell the user which section is absent
- Include only nodes/edges explicitly present in the report — no inference
- Only run `gemini` and `ls` — nothing else

---

## Step 0: Fix nano-banana skill path (prerequisite)

The skill is currently at `~/.claude/skills/SKILL.md` (directly in `skills/`,
no subdirectory). The correct location is `~/.claude/skills/nano-banana/SKILL.md`.

Actions:
1. Create `~/.claude/skills/nano-banana/` directory
2. Move `~/.claude/skills/SKILL.md` → `~/.claude/skills/nano-banana/SKILL.md`
3. Delete the old misplaced `~/.claude/~/.claude/` directory tree if it still exists

The skill content and frontmatter are already correct — only the path needs fixing.

---

## Critical files

- `~/.claude/agents/function-tracer.md` — defines the report format this agent parses
- `~/.claude/~/.claude/skills/nano-banana/SKILL.md` — defines the exact gemini CLI invocation pattern
- `~/.claude/agents/TEMPLATE.md` — frontmatter schema reference

---

## Verification

After implementation:
1. Run `function-tracer` on any function in a local project to produce a report
2. Pipe or paste that report to `function-diagram`
3. Confirm `./nanobanana-output/` contains a new image file
4. Confirm the diagram contains the expected nodes (central function, callers, callees)
