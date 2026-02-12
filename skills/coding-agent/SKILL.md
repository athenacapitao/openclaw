---
name: coding-agent
description: Run Codex CLI, Claude Code, OpenCode, or Pi Coding Agent via background process for programmatic control.
metadata:
  {
    "openclaw": { "emoji": "🧩", "requires": { "anyBins": ["claude", "codex", "opencode", "pi"] } },
  }
---

# Coding Agent (exec-first)

Use **exec** (with optional background mode) for all coding agent work. Simple and effective.

**Critical Note:** Always use the `exec` tool, NOT the `bash` tool. The exec tool handles parameters correctly.

## ⚠️ PTY Mode Required!

Coding agents (Codex, Claude Code, Pi) are **interactive terminal applications** that need a pseudo-terminal (PTY) to work correctly. Without PTY, you'll get broken output, missing colors, or the agent may hang.

**Always use `pty:true`** when running coding agents:

## ⚠️ CRITICAL: Never Use Systemd User Commands in Claude Code!

**Systemd user commands WILL FAIL** in Claude Code background PTY sessions because they require:

- `DBUS_SESSION_BUS_ADDRESS` environment variable
- `XDG_RUNTIME_DIR` environment variable

These are automatically set in interactive shells but NOT in background PTY sessions.

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

**Learned:** 2026-02-10 - Claude Code systemd bus issue when completing athena-tasks Phase 1

```bash
# ✅ Correct - with PTY
exec pty:true command:"codex exec 'Your prompt'"

# ❌ Wrong - no PTY, agent may break
exec command:"codex exec 'Your prompt'"
```

### Exec Tool Parameters

| Parameter    | Type    | Description                                                                 |
| ------------ | ------- | --------------------------------------------------------------------------- |
| `command`    | string  | The shell command to run                                                    |
| `pty`        | boolean | **Use for coding agents!** Allocates a pseudo-terminal for interactive CLIs |
| `workdir`    | string  | Working directory (agent sees only this folder's context)                   |
| `background` | boolean | Run in background, returns sessionId for monitoring                         |
| `timeout`    | number  | Timeout in seconds (kills process on expiry)                                |
| `elevated`   | boolean | Run on host instead of sandbox (if allowed)                                 |

### Process Tool Actions (for background sessions)

| Action      | Description                                          |
| ----------- | ---------------------------------------------------- |
| `list`      | List all running/recent sessions                     |
| `poll`      | Check if session is still running                    |
| `log`       | Get session output (with optional offset/limit)      |
| `write`     | Send raw data to stdin                               |
| `submit`    | Send data + newline (like typing and pressing Enter) |
| `send-keys` | Send key tokens or hex bytes                         |
| `paste`     | Paste text (with optional bracketed mode)            |
| `kill`      | Terminate the session                                |

---

## Quick Start: One-Shot Tasks

For quick prompts/chats, create a temp git repo and run:

```bash
# Quick chat (Codex needs a git repo!)
SCRATCH=$(mktemp -d) && cd $SCRATCH && git init && codex exec "Your prompt here"

# Or in a real project - with PTY!
exec pty:true workdir:~/Projects/myproject command:"codex exec 'Add error handling to the API calls'"
```

**Why git init?** Codex refuses to run outside a trusted git directory. Creating a temp repo solves this for scratch work.

---

## The Pattern: workdir + background + pty

For longer tasks, use background mode with PTY:

```bash
# Ensure directory exists (CRITICAL)
mkdir -p ~/project

# Start agent in target directory (with PTY!)
exec pty:true workdir:~/project background:true command:"codex exec --full-auto 'Build a snake game'"
# Returns sessionId for tracking

# Monitor progress
process action:log sessionId:XXX

# Check if done
process action:poll sessionId:XXX

# Send input (if agent asks a question)
process action:write sessionId:XXX data:"y"

# Submit with Enter (like typing "yes" and pressing Enter)
process action:submit sessionId:XXX data:"yes"

# Kill if needed
process action:kill sessionId:XXX
```

**Why workdir matters:** Agent wakes up in a focused directory, doesn't wander off reading unrelated files (like your soul.md 😅).

---

## Codex CLI

**Model:** `gpt-5.2-codex` is the default (set in ~/.codex/config.toml)

### Flags

| Flag            | Effect                                             |
| --------------- | -------------------------------------------------- |
| `exec "prompt"` | One-shot execution, exits when done                |
| `--full-auto`   | Sandboxed but auto-approves in workspace           |
| `--yolo`        | NO sandbox, NO approvals (fastest, most dangerous) |

### Building/Creating

```bash
# Ensure directory exists first!
mkdir -p ~/project

# Quick one-shot (auto-approves) - remember PTY!
exec pty:true workdir:~/project command:"codex exec --full-auto 'Build a dark mode toggle'"

# Background for longer work
exec pty:true workdir:~/project background:true command:"codex --yolo 'Refactor the auth module'"
```

### Reviewing PRs

**⚠️ CRITICAL: Never review PRs in OpenClaw's own project folder!**
Clone to temp folder or use git worktree.

```bash
# Clone to temp for safe review
REVIEW_DIR=$(mktemp -d)
git clone https://github.com/user/repo.git $REVIEW_DIR
cd $REVIEW_DIR && gh pr checkout 130
exec pty:true workdir:$REVIEW_DIR command:"codex review --base origin/main"
# Clean up after: trash $REVIEW_DIR

# Or use git worktree (keeps main intact)
git worktree add /tmp/pr-130-review pr-130-branch
exec pty:true workdir:/tmp/pr-130-review command:"codex review --base main"
```

### Batch PR Reviews (parallel army!)

```bash
# Ensure directory exists first!
mkdir -p ~/project

# Fetch all PR refs first
git fetch origin '+refs/pull/*/head:refs/remotes/origin/pr/*'

# Deploy the army - one Codex per PR (all with PTY!)
exec pty:true workdir:~/project background:true command:"codex exec 'Review PR #86. git diff origin/main...origin/pr/86'"
exec pty:true workdir:~/project background:true command:"codex exec 'Review PR #87. git diff origin/main...origin/pr/87'"

# Monitor all
process action:list

# Post results to GitHub
gh pr comment <PR#> --body "<review content>"
```

---

## Claude Code

**Recommended Workflow (tested 2026-02-10):**

```bash
# 0. Ensure directory exists (CRITICAL)
mkdir -p /path/to/project

# 1. Start Claude Code in target directory (interactive mode)
exec pty:true workdir:/path/to/project background:true command:"claude"
# Returns sessionId for tracking

# 2. Interact with Claude Code via process tool
process action:write sessionId:XXX data:"Create a file explaining hello world"
process action:submit sessionId:XXX data:"yes"  # Send + Enter

# 3. Monitor progress
process action:log sessionId:XXX

# 4. When done, kill the session
process action:kill sessionId:XXX
```

**For one-shot commands:**

```bash
# Ensure directory exists first!
mkdir -p ~/project

# Quick one-liner (non-interactive, prints output)
exec pty:true workdir:~/project command:"claude --print 'Your task'"
```

**Key Workflow Rules:**

- **Always start in project directory** - `workdir:/path/to/project`
- **Use `pty:true`** - Claude Code needs a pseudo-terminal
- **Interactive mode** - Run just `claude` without flags for full interaction
- **Chain commands** - Can run multiple git commands: `git add file && git commit -m "msg" && git push`
- **Use for coding projects** - Building features, apps, tools, GitHub operations

**Top 5 Essential Claude Code Commands:**

1. **`/compact`** — Compress Your Context
   - **WHY:** Claude Code has a finite context window. As your conversation grows, Claude's performance degrades.
   - **WHEN:** Every 15-20 minutes on complex tasks, or before starting a new sub-task
   - **HOW:** Just type `/compact` during a session

2. **`/clear`** — Start Fresh
   - **WHY:** Leftover context from unrelated tasks causes "context rot"
   - **WHEN:** Every time you switch to a new, unrelated task, or when Claude seems confused
   - **HOW:** Just type `/clear`

3. **`/init`** — Set Up Your Project Memory
   - **WHY:** Creates (or updates) a CLAUDE.md file in your project root — Claude's memory of your project
   - **WHEN:** First time using Claude Code in a project, or after major architectural changes
   - **HOW:** Just type `/init`

4. **`/model`** — Switch Models on the Fly
   - **WHY:** Not every task needs your most expensive model. Optimize for speed/quality.
   - **WHEN:**
     - Switch to Haiku for quick, repetitive tasks (generating tests, formatting)
     - Switch to Opus for complex reasoning, architecture decisions, or tricky debugging
   - **HOW:** Type `/model` and select from available models (Sonnet, Haiku, Opus)

5. **`/help`** — Discover Everything Available
   - **WHY:** Shows all your custom commands from `.claude/commands/`, skills from `.claude/skills/`, and MCP servers
   - **WHEN:** When you're new to Claude Code, or after installing plugins/commands
   - **HOW:** Just type `/help`

**Bonus Tips:**

- Shift+Tab → Toggle between permission modes
- Tab → Toggle thinking visibility
- # followed by text → Quickly add to Claude's memory
- Cmd+P → Model picker
- Full docs: code.claude.com/docs/en/slash-commands

---

## OpenCode

```bash
mkdir -p ~/project
exec pty:true workdir:~/project command:"opencode run 'Your task'"
```

---

## Pi Coding Agent

```bash
# Install: npm install -g @mariozechner/pi-coding-agent
mkdir -p ~/project
exec pty:true workdir:~/project command:"pi 'Your task'"

# Non-interactive mode (PTY still recommended)
exec pty:true command:"pi -p 'Summarize src/'"

# Different provider/model
exec pty:true command:"pi --provider openai --model gpt-4o-mini -p 'Your task'"
```

**Note:** Pi now has Anthropic prompt caching enabled (PR #584, merged Jan 2026)!

---

## Parallel Issue Fixing with git worktrees

For fixing multiple issues in parallel, use git worktrees:

```bash
# 1. Create worktrees for each issue
git worktree add -b fix/issue-78 /tmp/issue-78 main
git worktree add -b fix/issue-99 /tmp/issue-99 main

# 2. Launch Codex in each (background + PTY!)
exec pty:true workdir:/tmp/issue-78 background:true command:"pnpm install && codex --yolo 'Fix issue #78: <description>. Commit and push.'"
exec pty:true workdir:/tmp/issue-99 background:true command:"pnpm install && codex --yolo 'Fix issue #99: <description>. Commit and push.'"

# 3. Monitor progress
process action:list
process action:log sessionId:XXX

# 4. Create PRs after fixes
cd /tmp/issue-78 && git push -u origin fix/issue-78
gh pr create --repo user/repo --head fix/issue-78 --title "fix: ..." --body "..."

# 5. Cleanup
git worktree remove /tmp/issue-78
git worktree remove /tmp/issue-99
```

---

## Sequential Task Execution Workflow (CRITICAL - Updated 2026-02-11)

**Rule:** Process tasks ONE AT A TIME, sequentially. No parallel execution of multiple tasks.

**For Each Task:**

1. **Start Claude Code** in project directory

   ```bash
   mkdir -p /path/to/project
   exec pty:true workdir:/path/to/project background:true command:"claude"
   ```

2. **Send task to Claude Code** with clear task ID and requirements

   ```bash
   process action:write sessionId:XXX data:"Task ID: P1-01. [Full task description]"
   process action:submit sessionId:XXX
   ```

3. **Monitor progress** periodically via process log

   ```bash
   process action:log sessionId:XXX
   ```

4. **Verify completion** - Check Claude Code's output and confirm all criteria met

5. **Push to GitHub** if task completed successfully

   ```bash
   # Via Claude Code or direct git commands
   git add .
   git commit -m "Task P1-01: [Brief description]"
   git push
   ```

6. **Notify Wilson via Telegram** with task status
   - Use `message` tool with `action:send`, `to:538939197`
   - Include task ID, completion status, and brief summary
   - If error occurred, include error details

7. **Kill Claude Code session**

   ```bash
   process action:kill sessionId:XXX
   ```

8. **Proceed to next task** - Only after current task is verified, pushed, and notified

**Critical Rules:**

- ✅ **One task at a time** - Never start the next task until current task is complete
- ✅ **Maintain overall context** - Always keep the big picture and project plan in mind
- ✅ **Verify before moving on** - Don't rush; check everything works
- ✅ **Notify for each task** - Wilson needs to know what's happening, task by task
- ❌ **No parallel execution** - Do NOT start multiple Claude Code sessions for different tasks
- ❌ **No skipping verification** - Always confirm the task is actually done before pushing

**Telegram Notification Format:**

**Success:**

```
✅ Task P1-01 completed & verified: [Brief description of what was done]
```

**Error:**

```
❌ Task P1-01 failed: [Error description]
```

**Why this matters:**

- Clear visibility into progress
- Early detection of issues
- Prevents cascading failures
- Wilson stays informed and can intervene if needed

---

## ⚠️ Rules

1. **Always use pty:true** - coding agents need a terminal!
2. **Always use exec tool, not bash** - `exec pty:true workdir:/path/to/project`
3. **Always create directory first** - `mkdir -p /path/to/project` before running exec
4. **Respect tool choice** - if user asks for Codex, use Codex.
   - Orchestrator mode: do NOT hand-code patches yourself.
   - If an agent fails/hangs, respawn it or ask the user for direction, but don't silently take over.
5. **Be patient** - don't kill sessions because they're "slow"
6. **Monitor with process:log** - check progress without interfering
7. **--full-auto for building** - auto-approves changes
8. **vanilla for reviewing** - no special flags needed
9. **Parallel is OK** - run many Codex processes at once for batch work
10. **NEVER start Codex in ~/clawd/** - it'll read your soul docs and get weird ideas about the org chart!
11. **NEVER checkout branches in ~/Projects/openclaw/** - that's the LIVE OpenClaw instance!

---

## Progress Updates (Critical)

When you spawn coding agents in the background, keep the user in the loop.

- Send 1 short message when you start (what's running + where).
- Then only update again when something changes:
  - a milestone completes (build finished, tests passed)
  - the agent asks a question / needs input
  - you hit an error or need user action
  - the agent finishes (include what changed + where)
- If you kill a session, immediately say you killed it and why.

This prevents the user from seeing only "Agent failed before reply" and having no idea what happened.

---

## Auto-Notify on Completion

For long-running background tasks, append a wake trigger to your prompt so OpenClaw gets notified immediately when the agent finishes (instead of waiting for the next heartbeat):

```
... your task here.

When completely finished, run this command to notify me:
openclaw gateway wake --text "Done: [brief summary of what was built]" --mode now
```

**Example:**

```bash
mkdir -p ~/project
exec pty:true workdir:~/project background:true command:"codex --yolo exec 'Build a REST API for todos.

When completely finished, run: openclaw gateway wake --text \"Done: Built todos REST API with CRUD endpoints\" --mode now'"
```

This triggers an immediate wake event — Skippy gets pinged in seconds, not 10 minutes.

---

## Learnings (Jan 2026)

- **PTY is essential:** Coding agents are interactive terminal apps. Without `pty:true`, output breaks or agent hangs.
- **Git repo required:** Codex won't run outside a git directory. Use `mktemp -d && git init` for scratch work.
- **Directory must exist:** Always run `mkdir -p /path/to/project` before using exec.
- **Use exec tool:** `exec pty:true workdir:/path/to/project command:"..."` - NOT bash.
- **exec is your friend:** `codex exec "prompt"` runs and exits cleanly - perfect for one-shots.
- **submit vs write:** Use `submit` to send input + Enter, `write` for raw data without newline.
- **Sass works:** Codex responds well to playful prompts. Asked it to write a haiku about being second fiddle to a space lobster, got: _"Second chair, I code / Space lobster sets the tempo / Keys glow, I follow"_ 🦞

---

## Development Server Management

When working on projects with a development server (like athena-tasks), you'll need to restart it frequently during testing.

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
- No SIGTERM confusion in exec output

### Using Helper Scripts (Recommended)

For projects with frequent server restarts, create a `restart-server.sh` helper:

```bash
#!/bin/bash
pgrep -f "node server.js" > /dev/null && pkill -f "node server.js" 2>/dev/null
sleep 1
node server.js &
```

Make it executable and use it:

```bash
chmod +x restart-server.sh
./restart-server.sh
```

### Important Note on SIGTERM Messages

When using exec tool, you may see:

```
Command aborted by signal SIGTERM
```

**This is normal!** The SIGTERM is about the **shell wrapper** terminating, NOT the node server itself. If:

- Server restarts successfully
- No "port already in use" errors
- API calls work after restart

...then everything is fine. Ignore the SIGTERM message.

### Testing Server After Restart

```bash
# Quick health check
curl -s http://127.0.0.1:7700/api/health

# Or use background session to monitor logs
process action:log sessionId:XXX
```

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

**Documentation:** `/home/athena/.openclaw/workspace/athena-tasks/LESSONS.md` (full examples)

---

## 🚀 Claude Code: Relentless Execution Mode (Updated 2026-02-11)

**PHILOSOPHY:** Never stop. Never give up. The goal is absolute completion. Be relentless. Be proactive. Anticipate problems before they happen.

### High-Performance Workflow

```bash
# Step 1: Launch Claude Code
mkdir -p /path/to/project
exec pty:true workdir:/path/to/project background:true command:"claude"

# Step 2: Send ruthless task - ALL requirements upfront
process action:write sessionId:XXX data:"Task: [Clear goal]. Requirements: [bullet list]. Work autonomously. Test everything. Do not ask trivial questions. Keep pushing until done."
process action:submit sessionId:XXX

# Step 3: AGGRESSIVE monitoring - check every 10-15 seconds
process action:log sessionId:XXX

# Step 4: If Claude stalls - nudge immediately
process action:write sessionId:XXX data:"Continue. Don't stop."
process action:submit sessionId:XXX

# Step 5: Only kill when TRULY finished (tested and working)
process action:kill sessionId:XXX
```

### Speed Optimization Rules

**1. Be Ruthless with Context**

- Use `/compact` every 15-20 minutes on long tasks
- Use `/clear` when switching to unrelated task types
- If response slows down or Claude forgets → `/compact` immediately

**2. Never Wait for Decisions**

- When Claude asks "Should I use X or Y?" → Decide instantly
- Use best judgment - pick the more robust option and keep moving
- If unsure, choose one and note it can be changed later

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

### Aggressive Monitoring

**Check intervals:**

- First 2 minutes: Every 10 seconds
- Next 10 minutes: Every 30 seconds
- After 12 minutes: Every 60 seconds

**Watch for:**

- 🟢 Good: Progress messages, code being written, no errors
- 🟡 Warning: Same message repeated, no output for 60s, asking "Should I...?"
- 🔴 Bad: Errors not fixed, looping, no output for 120s

**Respond:**

- 🟡 Yellow: Nudge → "Continue. Don't stop."
- 🔴 Red: Redirect → "You're stuck. Try this alternative approach: [X]"
- 🔴 Looping: `/clear` and restart with fresh approach

### Task Completion Verification

**Before killing Claude Code, verify:**

- [ ] All task requirements completed
- [ ] Tests pass (if applicable)
- [ ] Code compiles/builds without errors
- [ ] Functionality verified (manually tested)
- [ ] Git committed and pushed
- [ ] No critical bugs

**Only THEN kill:**

```bash
process action:kill sessionId:XXX
```

### When to Escalate

**Signal Wilson when:**

- Stuck for >5 minutes despite nudges
- Critical ambiguity affecting architecture
- Security concern (need approval)
- Approach debate (multiple valid paths)
- External dependency issue (can't fix)

### Full Documentation

See `/home/athena/.openclaw/workspace/CLAUDE-CODE-OPTIMIZED.md` for complete guide including:

- Detailed error recovery examples
- Sequential task workflow
- Common error fixes reference
- Proactive problem prevention patterns
