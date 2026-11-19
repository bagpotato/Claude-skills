---
name: scripting
description: >
  Use this skill whenever the user wants to write, generate, fix, or improve a script in Bash/Shell, Python, or JavaScript/Node.js.
  Trigger this skill when the user says things like "write me a script", "create a script", "make a script that...", "automate this with a script", "I need a bash/python/js script", or describes a task that could be solved by scripting.
  This skill is ESPECIALLY important when the user hasn't fully specified what they want — the skill mandates an interview phase BEFORE writing any code.
  Also trigger when the user gives feedback on a previously written script (e.g., "that's wrong", "it doesn't work", "you made a mistake") — the skill handles error tracking and revision.
---

# Scripting Skill

A skill for writing high-quality Bash/Shell, Python, and JavaScript/Node.js scripts.
The key principle: **always interview the user before writing**, and **never repeat a mistake the user has corrected**.

---

## Supported Languages

- **Bash / Shell** — automation, file operations, system tasks, cron jobs
- **Python** — data processing, APIs, file manipulation, automation
- **JavaScript / Node.js** — server scripts, file I/O, API calls, CLI tools

---

## Phase 1: Pre-Script Interview

Before writing a single line of code, ask the user the following questions. Group them in a single message — do not ask one at a time unless the user seems overwhelmed.

### Required Questions

1. **Purpose & Goal**
   - What should the script do? What problem does it solve?
   - What is the expected end result when it runs successfully?

2. **Inputs & Outputs**
   - What inputs does the script receive? (files, arguments, stdin, API responses, env vars, etc.)
   - What should it output? (files, stdout, return codes, side effects, etc.)
   - Are there edge cases to handle? (empty input, missing files, invalid formats, network errors, etc.)

3. **Environment**
   - What OS will this run on? (Linux, macOS, Windows/WSL)
   - Is there a specific shell, Python version, or Node version to target?
   - Any dependencies or libraries available (or to avoid)?

4. **Language** (if not already specified)
   - Preferred language: Bash, Python, or JavaScript?
   - Any style preferences? (verbose logging, minimal, defensive, etc.)

### Optional (ask if relevant)

- Should it be idempotent (safe to run multiple times)?
- Does it need error handling / exit codes?
- Should it be silent or verbose?
- Will it run interactively or as a background job?

---

## Phase 2: Writing the Script

Once the interview is complete:

- Write the full script, not a skeleton
- Include comments explaining non-obvious logic
- Add error handling appropriate to the language:
  - Bash: `set -euo pipefail`, trap errors
  - Python: try/except with meaningful messages
  - Node.js: try/catch, handle Promise rejections
- Match the target OS (e.g., avoid `gsed` on Linux, avoid `#!/bin/bash` on strict POSIX systems)
- Respect any mistake history (see Phase 3)

Present the script in a code block with the correct language tag.
After the script, briefly explain:
1. What it does (1–2 sentences)
2. How to run it (exact command)
3. Any assumptions made

---

## Phase 3: Mistake Tracking & Revision

This is the most critical part of the skill.

### When the user reports an error or gives corrective feedback:

1. **Acknowledge** the mistake clearly and specifically (do not be vague)
2. **Log it internally** in a "Mistake Memory" block — keep this updated across the conversation
3. **Fix** the script, addressing the root cause (not just the symptom)
4. **Confirm** the fix won't reintroduce the same pattern

### Mistake Memory Format

Maintain this block mentally and reference it before every revision or new script in the same conversation:

```
MISTAKE MEMORY
--------------
[#1] <Short description of the mistake>
     Trigger: what caused it
     Fix applied: what was changed
     Rule: never do X again in this conversation

[#2] ...
```

### Rules for Mistake Memory

- Never repeat a logged mistake in subsequent scripts or revisions
- If writing a second script in the same session, proactively avoid all logged mistakes
- If a new script is similar to a previous one, explicitly mention: "I remembered from earlier that [mistake], so I've made sure to [fix]"

---

## Phase 4: Iteration

After delivering a revised script:
- Ask if the fix resolved the issue
- Ask if there are any other issues or requirements to add
- Offer to add tests, a dry-run mode, or verbose logging if appropriate

---

## Style Defaults (unless user specifies otherwise)

| Language   | Defaults |
|------------|----------|
| Bash       | `#!/usr/bin/env bash`, `set -euo pipefail`, double-quote variables |
| Python     | Type hints where helpful, `if __name__ == "__main__"` guard, argparse for CLI |
| JavaScript | ES modules or CommonJS based on context, async/await, `process.exit()` on error |

---

## Anti-Patterns to Avoid (always)

- Do not write a script before completing the interview
- Do not write partial/skeleton scripts unless user explicitly asks
- Do not silently ignore errors in the script logic
- Do not repeat a mistake the user has already corrected
- Do not assume the OS without asking (e.g., don't use `brew` commands unless macOS is confirmed)
