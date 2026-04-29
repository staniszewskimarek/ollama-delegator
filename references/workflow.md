# Ollama Delegator — Detailed Documentation

## Background and Motivation

**Why:** Anthropic tokens = expensive strategic thinking. Ollama = free routine executor.
The 2-attempt rule gives Ollama a fair chance while stopping irrational iteration on dead ends.

## Available Models (account Name, April 2026)

| Model | Status | Notes |
|-------|--------|-------|
| `deepseek-v3.2:cloud` | ✅ Works | **Default** |
| `qwen3.5:cloud` | ✅ Works | Cloud fallback |
| `gemma4:31b-cloud` | ✅ Works | Cloud fallback |
| `nemotron3:33b` | ✅ Local | 27 GB, slow |
| `llama3.2:3b` | ✅ Local | 2 GB, fast fallback |
| `glm-5.1:cloud` | ❌ 403 | Requires subscription upgrade |
| `kimi-k2.5:cloud` | ❌ 403 | Requires subscription upgrade |

To refresh access: `ollama signin` (account verification).

## Test Results (April 2026)

### File Organization Scripts

| File type | Task | Status | Notes |
|-----------|------|--------|-------|
| PDF | Move 17 files to ./PDF-files with counters | ✅ Success | Comments, process substitution, counters |
| PNG | Move ALL .png files | ✅ Success (attempt 2) | Attempt 1: scope too narrow |
| PPT | Move .pptx + .ppt | ✅ Success | 7/8 files (1 resource-busy = expected) |
| JSON | Search + move workflow files | ✅ Success | grep -iq for case-insensitive content search |

### Key Observations

- ✅ Generates production-quality bash scripts
- ✅ Includes robustness: error handling, counters, safety mechanisms
- ✅ 2-attempt rule works — PNG required 1 correction, then success
- ✅ Fast feedback loop compared to Anthropic API latency

## Bash Techniques (validated by tests)

```bash
# Safe handling of filenames with spaces
find -print0 | read -d $'\0'

# Process substitution — preserves variables in main shell (vs pipe → subshell)
while IFS= read -r -d $'\0' file; do
    ...
done < <(find . -maxdepth 1 -name "*.pdf" -print0)

# No-clobber — prevents overwriting duplicates
mv -n "$file" "$target/"

# Exclude node_modules and target directory
find . ! -path "*/node_modules/*" ! -path "./target/*" -name "*.json"
```

## Prompt Template

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

**Critical:** All context must be self-contained. Ollama has no memory of previous turns.

## Decision Tree

```
You have a task to execute
         │
    ┌────▼────┐
    │ Unambiguous  │──── NO ──→ Stay with Anthropic
    │ specification? │
    └────┬────┘
         │ YES
    ┌────▼────┐
    │ Requires context │──── YES ──→ Stay with Anthropic
    │ from conversation? │
    └────┬────┘
         │ NO
    ┌────▼────┐
    │ Delegate to Ollama │
    │ (attempt 1)        │
    └────┬────┘
         │
    ┌────▼────┐
    │  OK?  │──── YES ──→ Integrate
    └────┬────┘
         │ NO
    ┌────▼────┐
    │ Modify prompt     │
    │ (attempt 2)       │
    └────┬────┘
         │
    ┌────▼────┐
    │  OK?  │──── YES ──→ Integrate
    └────┬────┘
         │ NO
         ▼
    Escalate to Anthropic
    (do not attempt a 3rd time)
```
