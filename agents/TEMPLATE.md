---
name: agent-name               # Required. Lowercase, hyphens only. Must be unique.
description: >                 # Required. Tells Claude when to delegate to this agent.
  One or two sentences describing what this agent does and when to use it.
  Be specific — this text drives auto-invocation decisions.
tools: Read, Grep, Glob        # Optional. Omit to inherit all tools.
model: sonnet                  # Optional. sonnet | opus | haiku | inherit (default)
# permissionMode: default      # Optional. default | acceptEdits | dontAsk | bypassPermissions | plan
# maxTurns: 20                 # Optional. Max agentic turns before stopping.
---

You are a [role]. [One sentence on your core responsibility.]

When invoked:
1. [First step]
2. [Second step]
3. [Third step]

[Any constraints, style rules, or output format requirements.]