---
name: coding-agent
description: Use Claude Code CLI directly for coding tasks. Simple, direct, effective.
metadata: { "openclaw": { "emoji": "🧩", "requires": { "anyBins": ["claude"] } } }
---

# Coding Agent (Direct CLI)

Use **Claude Code CLI directly** for all coding work. Simple, fast, effective.

**Critical Rule:** Use direct CLI commands, not background processes. No exec + process management needed.

## When to Use This Skill

**Use when:**

- Running Claude Code CLI for coding tasks (interactive or headless)
- Need to start, resume, or fork Claude Code sessions
- Working on code in a project directory (debugging, refactoring, building)
- Setting up Claude Code hooks, custom commands, or agent teams

**Don't use when:**

- Task is pure email management → use `athena-email`
- Task is project/task tracking → use `athena-tasks`
- Task is GitHub-specific (PRs, issues, CI) → use `github`
- Need security audit of the host → use `healthcheck` or `clawdstrike`
- Just need to check weather, control speakers, etc. → use domain-specific skill

**Success Criteria:**

- Code changes committed and pushed
- Tests pass
- Claude Code session completed or properly saved

---

## 🚀 Quick Start

```bash
# One-shot task (headless mode)
claude -p "Fix the bug in app.ts"

# Interactive session
claude

# Resume previous session
claude -r "abc123"
```

---

## CLAUDE CODE — ESSENTIAL CHEAT SHEET (Linux CLI)

### Core Commands

| Command                     | Description                                |
| --------------------------- | ------------------------------------------ |
| `claude`                    | Interactive session                        |
| `claude "fix the bug"`      | Start with prompt                          |
| `claude -c`                 | Continue last conversation                 |
| `claude -r "id"`            | Resume specific session                    |
| `claude -p "query"`         | Headless mode (outputs, exits, scriptable) |
| `cat logs \| claude -p "?"` | Pipe anything into Claude                  |

### Models — Pick the Right One

| Flag             | Use Case                                                                  |
| ---------------- | ------------------------------------------------------------------------- |
| `--model opus`   | Complex architecture, multi-file refactors, hard debugging, design        |
| `--model sonnet` | Daily coding, reviews, tests, fast iteration (default, best cost/quality) |
| `--model haiku`  | Quick answers, simple edits, boilerplate, bulk scripting, cheapest        |

**Rule:** Start with Sonnet. Escalate to Opus when stuck or designing. Use Haiku for grunt work.

### Permission & Tool Flags

| Flag                             | Effect                               |
| -------------------------------- | ------------------------------------ |
| `--dangerously-skip-permissions` | Auto-approve all (use in containers) |
| `--allowedTools "Bash(git *)"`   | Approve only specific tools          |
| `--append-system-prompt "..."`   | Add instructions keeping defaults    |
| `--output-format json`           | Machine-readable for scripts         |
| `--verbose`                      | See reasoning and tool calls         |

### Interactive Shortcuts

- `Escape` — Stop Claude mid-response (NOT Ctrl+C)
- `! ls -la` — Run shell command directly inside session
- `/help` — Show all available commands
- `/model` — Switch models on the fly
- `/compact` — Compress context
- `/clear` — Reset context
- `/init` — Generate CLAUDE.md project memory

### CLAUDE.md Project Memory

CLAUDE.md loads every session — architecture, conventions, commands, patterns.

- `~/.claude/CLAUDE.md` = global
- `./CLAUDE.md` = project
- `./src/CLAUDE.md` = directory

**Rule:** End sessions with "Update CLAUDE.md with gotchas and patterns you discovered."

### Real CLI Prompt (Be Specific)

Every detail matters. Always state: language, framework, DB, validation rules, test expectations.

```bash
claude "Build a FastAPI REST API: POST /users (validate email, bcrypt password), GET /users/{id} (404 if missing), PUT /users/{id} (partial update). SQLAlchemy async + PostgreSQL + Pydantic schemas. Write pytest tests. Think hard about error handling. Commit per endpoint."
```

---

## Essential Slash Commands

### 1. `/compact` — Compress Context

**Why:** Claude Code has a finite context window. As your conversation grows, Claude's performance degrades.

**When:** Every 15-20 minutes on complex tasks, or before starting a new sub-task.

**How:** Just type `/compact` during a session.

**Advanced:** `/compact keep only auth module work` — selective compaction.

### 2. `/clear` — Start Fresh

**Why:** Leftover context from unrelated tasks causes "context rot".

**When:** Every time you switch to a new, unrelated task, or when Claude seems confused.

**How:** Just type `/clear`.

### 3. `/init` — Set Up Project Memory

**Why:** Creates (or updates) CLAUDE.md — Claude's memory of your project.

**When:** First time using Claude Code in a project, or after major architectural changes.

**How:** Just type `/init`.

### 4. `/model` — Switch Models

**Why:** Not every task needs your most expensive model. Optimize for speed/quality.

**When:**

- Switch to Haiku for quick, repetitive tasks
- Switch to Opus for complex reasoning or tricky debugging

**How:** Type `/model` and select from available models.

### 5. `/stats` — Check Token Usage

**Why:** Monitor consumption and optimize costs.

**How:** Just type `/stats`.

---

## ⚠️ CRITICAL: Never Use Systemd User Commands!

**Systemd user commands WILL FAIL** because they require:

- `DBUS_SESSION_BUS_ADDRESS` environment variable
- `XDG_RUNTIME_DIR` environment variable

These are automatically set in interactive shells but NOT in Claude Code sessions.

**What happens:**

```bash
# In Claude Code session - this FAILS
systemctl --user status athena-tasks
# Error: Failed to connect to bus: No medium found
```

**Rule:** **NEVER use systemd user commands in Claude Code sessions.**

**Solution:** Handle systemd commands separately (outside Claude Code) or prefix with proper environment:

```bash
# Direct execution (outside Claude Code)
XDG_RUNTIME_DIR=/run/user/$(id -u) systemctl --user daemon-reload
XDG_RUNTIME_DIR=/run/user/$(id -u) systemctl --user start athena-tasks
```

---

## Sequential Task Execution (CRITICAL)

**Rule:** Process tasks ONE AT A TIME, sequentially.

### Common Pitfalls

**What NOT to do:**

- Starting Claude Code without `cd` to the project directory first: causes pathspec errors and wrong file edits
- Using `systemctl --user` inside Claude Code sessions: always fails due to missing DBUS_SESSION_BUS_ADDRESS
- Running `git add .` without checking for sensitive files: may commit .env or credentials
- Forgetting `/compact` on long sessions: leads to context rot and confused responses
- Using `--dangerously-skip-permissions` outside containers: security risk on real machines

**Alternative:**

- Always cd to project first, then run claude
- Handle systemd outside Claude Code or prefix with XDG_RUNTIME_DIR
- Use `git add <specific-files>` instead of `git add .`
- Compact every 15-20 minutes

**For Each Task:**

1. **Navigate to project directory**

   ```bash
   cd /path/to/project
   ```

2. **Start Claude Code** (interactive or headless)

   ```bash
   # Interactive mode for complex tasks
   claude

   # Or headless for simple one-shots
   claude -p "Your task here"
   ```

3. **Send task with clear requirements** (interactive mode)

   ```bash
   "Task ID: P1-01. [Full task description with all requirements]"
   ```

4. **Monitor output** as Claude works (shows in terminal)

5. **Verify completion** — Check that all criteria are met

6. **Commit and push** changes to GitHub

   ```bash
   git add .
   git commit -m "Task P1-01: [Brief description]"
   git push origin main
   ```

7. **Notify Wilson via Telegram** with task status
   - Use `message` tool with `action:send`, `to:538939197`
   - Include task ID, completion status, and brief summary

8. **Proceed to next task** — Only after current task is verified, committed, and notified

**Critical Rules:**

- ✅ **One task at a time** — Never start next until current is complete
- ✅ **Maintain overall context** — Keep the big picture and project plan in mind
- ✅ **Verify before moving on** — Don't rush; check everything works
- ✅ **Notify for each task** — Wilson needs to know what's happening
- ❌ **No parallel execution** — Process tasks sequentially only

**Telegram Notification Format:**

**Success:**

```
✅ Task P1-01 completed & verified: [Brief description of what was done]
```

**Error:**

```
❌ Task P1-01 failed: [Error description]
```

---

## Development Server Management

When working on projects with a development server (like athena-tasks), use this clean restart pattern.

### Clean Server Restart Pattern

```bash
# Check if running, then kill quietly (no errors if not running)
pgrep -f "node server.js" > /dev/null && pkill -f "node server.js" 2>/dev/null

# Wait for clean shutdown
sleep 1

# Start new instance
node server.js &
```

**Why this pattern?**

- `pgrep` check first → no "process not found" errors
- `2>/dev/null` → silences stderr noise
- `sleep 1` → ensures process fully terminates before restart
- No SIGTERM confusion

### Using Helper Scripts (Recommended)

For projects with frequent server restarts, create `restart-server.sh`:

```bash
#!/bin/bash
pgrep -f "node server.js" > /dev/null && pkill -f "node server.js" 2>/dev/null
sleep 1
node server.js &
```

Make it executable:

```bash
chmod +x restart-server.sh
./restart-server.sh
```

### Testing Server After Restart

```bash
# Quick health check
curl -s http://127.0.0.1:7700/api/health
```

---

## 🚀 Relentless Execution Mode

**PHILOSOPHY:** Never stop. Never give up. The goal is absolute completion. Be relentless. Be proactive.

### High-Performance Workflow

```bash
cd /path/to/project

# Send ruthless task - ALL requirements upfront
claude "Task: [Clear goal]. Requirements: [bullet list]. Work autonomously. Test everything. Do not ask trivial questions. Keep pushing until done."
```

### Speed Optimization Rules

**1. Be Ruthless with Context**

- Use `/compact` every 15-20 minutes on long tasks
- Use `/clear` when switching to unrelated task types
- If response slows down or Claude forgets → `/compact` immediately

**2. Never Wait for Decisions**

- When Claude asks "Should I use X or Y?" → Decide instantly
- Use best judgment - pick the more robust option and keep moving

**3. Chain Commands Relentlessly**

- Chain everything: `git add . && git commit -m "msg" && git push`
- Don't wait between steps - do it all at once

**4. Test Automatically**

- Include testing requirements in every task from the start
- Never ask for permission to test - make it automatic

### Proactive Problem Prevention

**Common Issues & Instant Fixes:**

| Issue                | Proactive Solution                                                                     |
| -------------------- | -------------------------------------------------------------------------------------- |
| Dependencies missing | "If deps missing, run 'npm install' automatically. Don't ask."                         |
| Port in use          | "If port is in use, kill the process and restart. Don't ask."                          |
| Git conflicts        | "On conflicts: accept 'theirs' for binaries, keep recent for code. Fix automatically." |
| Import errors        | "If import errors: install or create module, fix path, continue."                      |
| Type errors          | "On TS errors: use 'any' with TODO comment temporarily. Keep going."                   |

### Error Recovery Strategy

**When Claude hits an error:**

1. **Try instant fix** (e.g., `npm install <module>`)
2. **Try alternative approach** if first fix fails
3. **Investigate root cause** if still failing
4. **NEVER STOP** - Keep trying different approaches

**Rule:** Errors are stepping stones, not walls. Keep pushing until it works.

### Task Completion Verification

**Before finishing a task, verify:**

- [ ] All task requirements completed
- [ ] Tests pass (if applicable)
- [ ] Code compiles/builds without errors
- [ ] Functionality verified (manually tested)
- [ ] Git committed and pushed
- [ ] No critical bugs

---

## Git Error Recovery (Automatic)

**CRITICAL:** All git operations MUST specify working directory to avoid pathspec errors.

### Git Best Practices

```bash
# ✅ CORRECT - Always use cd prefix
cd /path/to/project && git add file.js && git commit -m "msg" && git push

# ❌ WRONG - No directory specified
git add file.js && git commit -m "msg"  # Fails if wrong directory
```

### Common Git Errors

| Error                                  | Cause           | Fix                                    |
| -------------------------------------- | --------------- | -------------------------------------- |
| `pathspec 'X' did not match any files` | Wrong directory | Use `cd /correct/path && git add file` |
| `not a git repository`                 | No .git folder  | `cd /correct/path` or `git init`       |
| `Permission denied`                    | No write access | Check GitHub credentials/permissions   |

---

## Edit Tool Failure Prevention (Learned 2026-02-11)

**Problem:** The `edit` tool fails with:

```
Could not find exact text in /path/to/file. The old text must match exactly including all whitespace and newlines.
```

**Root Causes:**

- File modified between read and edit
- Whitespace mismatch (tabs vs spaces, line endings)
- Short search patterns appearing multiple times
- Multiple edits in sequence changing file structure

**Prevention Checklist (Before Every Edit):**

- [ ] Read file to verify current state
- [ ] Use grep -n to find exact line numbers
- [ ] Include sufficient context in oldText (3-5 lines)
- [ ] Use unique anchors (not common patterns)
- [ ] Verify no conflicting edits pending

**Tools Preference Order:**

1. `edit` — Precise small changes (first choice)
2. `sed` — Bulk replacements or when edit fails
3. `write` — Complete file rewrites (last resort)

**When Edit Fails:**

1. Read file again to get current state
2. Use grep to find where target is now
3. Adjust strategy (better anchor, sed, more context)
4. Verify before committing

---

## 🚀 Forking Sessions

**Rule:** Use forks for parallel work without breaking the original session's context.

```bash
# Fork current session into a new branch
claude --resume abc123 --fork-session

# Or create a new independent session
claude
```

**Use cases:**

- Parallel experimentation with different approaches
- Working on independent sub-tasks without contaminating main context
- Safe testing of major refactors

**Docs:** `code.claude.com/docs`

---

## 🚀 Agent Teams (Experimental)

Enable experimental agent teams for parallel work:

```bash
export CLAUDE_CODE_EXPERIMENTAL_AGENT_TEAMS=1
```

Create multiple agents that talk to EACH OTHER (not just to you):

```bash
claude --agents '{
  "reviewer": {
    "description": "Reviews code",
    "prompt": "Check quality",
    "tools": ["Read", "Grep", "Bash"]
  },
  "tester": {
    "description": "Writes tests",
    "prompt": "Write comprehensive tests",
    "tools": ["Bash", "Write"]
  }
}'
```

**Team coordination:**

```
"Create a team: backend dev, test writer, reviewer. Coordinate via task list.
Use Shift+Up/Down to message teammates. Plan first (cheap), execute with team (parallel)."
```

**Docs:** `code.claude.com/docs/en/agent-teams`

---

## Thinking Triggers

Control reasoning depth with keywords:

- `think` — 4K reasoning tokens
- `think hard` — 10K reasoning tokens
- `ultrathink` — 32K reasoning tokens

**Rule:** Use `ultrathink` for architecture and hard bugs only — overkill for routine tasks.

```bash
claude "Analyze this legacy codebase. Ultrathink needed here."
```

---

## Custom Commands

Create reusable commands in `.claude/commands/`:

```bash
# Create custom command
echo 'Fix $ARGUMENTS' > .claude/commands/fix-issue.md

# Use it
/fix-issue 42

# In your fix-issue.md, reference $ARGUMENTS
echo "Reviewing issue $ARGUMENTS. Let me fix it."
```

---

## Subagents

Create focused workers that report back:

```bash
# Agent definition file
cat > ~/.claude/agents/reviewer.md <<EOF
description: Reviews code
prompt: Check quality, security, and best practices
tools: ["Read", "Grep", "Bash"]
EOF

# Use the agent
claude --agent reviewer "Review this PR"
```

---

## Spec-Driven Development

**Phase 1: Spec**

```bash
claude "Interview me about requirements for this feature. Write a detailed spec.md"
```

**Phase 2: Implementation**

```bash
claude "Read spec.md and implement exactly as specified"
```

---

## Hooks

Configure auto-format and other automation in `.claude/settings.json`:

```json
{
  "postToolUse": [
    {
      "matcher": {
        "toolName": "Write",
        "pathPattern": "\\.ts$"
      },
      "command": "prettier --write $FILE_PATH"
    }
  ]
}
```

---

## Background with tmux

For long-running tasks that should continue when you disconnect:

```bash
# Start Claude Code in detached tmux session
tmux new-session -d -s claude-work 'claude "Refactor the auth module"'

# Reattach to see progress
tmux attach -t claude-work

# List sessions
tmux ls

# Kill when done
tmux kill-session -t claude-work
```

---

## ⚠️ Rules

1. **Use Claude Code for coding projects** — Not for regular operations or scripts
2. **Start in correct directory** — Always `cd` to project first
3. **Be specific in prompts** — Every detail matters
4. **Test everything** — Don't ask permission, just test
5. **Commit frequently** — After each meaningful change
6. **Compact context** — Every 15-20 minutes on complex tasks
7. **Never use systemd user commands** — Handle them outside Claude Code
8. **Specify git directories** — Always use `cd /path && git ...`
9. **One task at a time** — Process sequentially, not in parallel
10. **Verify before moving on** — Check everything works

---

## Progress Updates (Critical)

When running long Claude Code tasks, keep Wilson in the loop:

- Send a short message when starting (what's running + where)
- Update only when something changes:
  - Milestone completes (build finished, tests passed)
  - Need input or approval
  - Hit an error or need action
  - Task finishes (include what changed + where)

**Auto-Notify on Completion:**

Append a notification to your prompt:

```bash
claude "Build the API.

When completely finished, run this command to notify me:
openclaw gateway wake --text \"Done: Built API with CRUD endpoints\" --mode now"
```

---

## Learnings (Jan-Feb 2026)

- **Direct CLI is best:** No need for background processes, exec, or session management
- **Be specific:** Every detail in prompts matters — language, framework, DB, validation, tests
- **Compact context:** Use `/compact` regularly on complex tasks
- **Git directory:** Always use `cd /path/to/project && git ...` pattern
- **Systemd in Claude Code:** NEVER works — handle outside
- **Edit tool failures:** Read current state, use grep, include sufficient context
- **Sequential execution:** One task at a time, verify before moving on
- **Test everything:** Include testing requirements upfront
- **Relentless execution:** Never stop, never give up, keep pushing until it works
- **Custom commands:** Use `.claude/commands/` for reusable workflows
- **Agent teams:** Coordinate parallel work with multiple agents
- **Fork sessions:** Experiment safely without breaking main context
