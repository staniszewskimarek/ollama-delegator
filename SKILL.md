---
name: ollama-delegator
description: This skill should be used when the user asks to "delegate to ollama", "use ollama pattern", "run through ollama", "save anthropic tokens", "ollama executor", "delegate task to local model", or wants to execute a task via Ollama before escalating to Anthropic. Implements the Plan → Delegate → Verify workflow with a strict 2-attempt limit.
version: 1.0.0
allowed-tools: Bash
---

# Ollama Pattern — Plan → Delegate → Verify

Delegate routine tasks to Ollama to save Anthropic tokens. Maximum 2 attempts — after the second failure, escalate to Anthropic without further iteration.

## Default Model

```
deepseek-v3.2:cloud
```

Local fallback: `llama3.2:3b` or `nemotron3:33b`.

> Note: `glm-5.1:cloud` and `kimi-k2.5:cloud` require a higher subscription tier (403 for current account).

## Procedure

### Step 1 — PLAN (Anthropic)

Before delegating, verify:

- Is the task unambiguously specified?
- Does the prompt contain ALL needed context? (Ollama has no memory of previous turns)
- Is this a delegation candidate? (see table below)

If yes — write a self-contained prompt and proceed to step 2.

### Step 2 — DELEGATE (Ollama)

```bash
ollama run deepseek-v3.2:cloud "PROMPT"
```

Structure the prompt as:

```
CONTEXT:
[Problem description, goal, constraints]

SPECIFICATION:
[Exact requirements, format, edge cases]

CODE/REFERENCE (optional):
[Example code if applicable]

TASK:
Generate [output type] meeting the requirements above.
```

**Attempt 2:** If attempt 1 failed — modify the prompt and run again.

**After 2 failures: STOP. Escalate to Anthropic. Do not iterate further.**

### Step 3 — VERIFY (Anthropic)

- Read the output
- Test the code if applicable
- OK → integrate
- FAIL attempt 1 → modify prompt, attempt 2
- FAIL attempt 2 → escalate to Anthropic

## When to Delegate ✅

| Task | Example |
|------|---------|
| Implementation from clear spec | "write a bash script that..." |
| Tests, docs, comments | "add docstrings to function X" |
| Refactoring with explicit rules | "replace tabs with spaces in file" |
| Generating test data | "generate 10 sample JSON objects" |
| Debugging an obvious error | "fix SyntaxError on line 42" |
| First drafts before involving Anthropic | "write a draft sort function" |

## When NOT to Delegate ❌

| Situation | Reason |
|-----------|--------|
| Multi-step reasoning required | Ollama has no conversation context |
| Architectural or design decision | Requires deeper analysis |
| Non-obvious error | Needs diagnostics |
| Code going to production without review | Too high risk |
| Ollama already failed 2 times | Escalate, do not iterate |

## Additional Resources

- **`references/workflow.md`** — detailed docs, test results, bash techniques, decision tree

## Quick Start Example

Task: move 17 PDF files to ./PDF-files with counters

```bash
ollama run deepseek-v3.2:cloud "
CONTEXT: I need a bash script to organize files.

SPECIFICATION:
- Find all .pdf files in the current directory (non-recursive)
- Move them to ./PDF-files (create if missing)
- Print a counter: 'Moved X/Y files'
- Use mv -n (no-clobber)
- Handle filenames with spaces safely

TASK: Generate a ready-to-run bash script."
```
