---
name: gemini
description: Gemini CLI for one-shot Q&A, summaries, and generation.
homepage: https://ai.google.dev/
metadata:
  {
    "openclaw":
      {
        "emoji": "♊️",
        "requires": { "bins": ["gemini"] },
        "install":
          [
            {
              "id": "brew",
              "kind": "brew",
              "formula": "gemini-cli",
              "bins": ["gemini"],
              "label": "Install Gemini CLI (brew)",
            },
          ],
      },
  }
---

# Gemini CLI

Use Gemini in one-shot mode with a positional prompt (avoid interactive mode).

## When to Use This Skill

**Use when:**

- Need a quick one-shot answer from Gemini AI
- Generating text, summaries, or structured output via Gemini
- Using Gemini extensions for specialized tasks

**Don't use when:**

- Need coding agent workflow → use `coding-agent`
- Need to interact with GitHub → use `github`
- Need persistent conversation → Gemini CLI is one-shot only

**Success Criteria:**

- Gemini returns expected output format
- Auth is configured (if needed)

Quick start

- `gemini "Answer this question..."`
- `gemini --model <name> "Prompt..."`
- `gemini --output-format json "Return JSON"`

Extensions

- List: `gemini --list-extensions`
- Manage: `gemini extensions <command>`

Notes

- If auth is required, run `gemini` once interactively and follow the login flow.
- Avoid `--yolo` for safety.

### Common Pitfalls

**What NOT to do:**

- Using interactive mode when one-shot works: wastes time and context
- Using `--yolo` flag: bypasses safety checks
- Forgetting to specify `--model` for specific model needs: uses default
