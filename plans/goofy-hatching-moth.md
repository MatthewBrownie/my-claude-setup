# Plan: function-diagram Agent Fallback Chain

## Context
The function-diagram agent currently only calls the Gemini CLI (nano-banana). If that tool is unavailable or fails, the agent has no recovery path. Adding two ordered fallbacks ensures a diagram is always produced.

## File to Modify
`C:\Users\mbrow\.claude\agents\function-diagram.md`

## Fallback Chain

| Priority | Renderer | Output | Availability |
|----------|----------|--------|-------------|
| 1 (primary) | Gemini CLI (`gemini`) | Image via nano-banana | Requires Gemini CLI + key |
| 2 (secondary) | Graphviz (`dot`) | PNG/SVG image | `brew/apt install graphviz` |
| 3 (tertiary) | Mermaid text | `.mmd` file | Always works; rendered in VS Code / GitHub |

## Changes

### 1. Expand `tools` frontmatter
Current: `tools: Bash(gemini:*)`
New: `tools: Bash(gemini:*, which:*, dot:*, mmdc:*, ls:*)`

This allows the agent to probe tool availability with `which` and run `dot` or `mmdc`.

### 2. Restructure Step 3 into a three-attempt sequence

**Attempt 1 — Gemini CLI** (unchanged)
```bash
gemini --yolo "/diagram '[PROMPT]' --type='call-graph' --style='technical'"
```
On success → proceed to Step 4 (report output).
On failure (non-zero exit or error output) → attempt 2.

**Attempt 2 — Graphviz**
- The agent generates DOT language inline from the parsed callers/callees/data-flow.
- DOT node shapes: `box` for functions, `cylinder` for databases, `ellipse` for network.
- Runs: `echo '[DOT_SOURCE]' | dot -Tpng -o ./function-diagram-output/[FUNCTION_NAME].png`
- Creates output directory if absent: `mkdir -p ./function-diagram-output`
- On success → present the output path.
- On failure (dot not found) → attempt 3.

**Attempt 3 — Mermaid**
- The agent generates `graph LR` Mermaid syntax from the same parsed data.
- Writes to `./function-diagram-output/[FUNCTION_NAME].mmd`
- If `mmdc` is available (`which mmdc`), also renders to PNG.
- Always succeeds as a text file. Tells the user how to view it (VS Code Mermaid Preview, GitHub markdown fence).

### 3. Update Step 4 to describe the actual output format used
Report which renderer succeeded and show the output path or file contents accordingly.

### 4. Update Constraints
- Expand allowed commands to match the new tool list.
- Add: "Attempt renderers in order. Do not skip to a later fallback without first trying earlier ones."
- Add: "The Mermaid fallback must always succeed — if writing the file fails, output the Mermaid syntax directly in the response."

## Verification
- Invoke the agent with a sample function-tracer report.
- Confirm it tries Gemini first.
- To test fallbacks: temporarily rename `gemini` and `dot` from PATH scope and confirm it degrades gracefully through to Mermaid output.
