  # ollama-delegator

  Delegate routine tasks to Ollama before escalating to Anthropic. Saves tokens on repetitive work.

  ## How it works

  Plan → Delegate → Verify with a strict 2-attempt limit.

  1. **Plan** (Anthropic) — define the task, write a self-contained prompt
  2. **Delegate** (Ollama) — run via `ollama run deepseek-v3.2:cloud "PROMPT"`
  3. **Verify** (Anthropic) — check output, integrate or escalate

  After 2 failed attempts — stop and handle it in Anthropic. No third attempt.

  ## Installation (Claude Code)

  ```bash
  mkdir -p ~/.claude/skills/ollama-delegator/references
  cp SKILL.md ~/.claude/skills/ollama-delegator/
  cp references/workflow.md ~/.claude/skills/ollama-delegator/references/

  Then invoke with /ollama-delegator or let Claude trigger it automatically.

  Default model

  deepseek-v3.2:cloud — fast, production-quality output, free tier.

  When to delegate

  ✅ Implementation from clear spec, tests, docs, refactoring, test data generation
  ❌ Multi-step reasoning, architectural decisions, non-obvious bugs, production code without review
