---
name: github
description: "Interact with GitHub using the `gh` CLI. Use `gh issue`, `gh pr`, `gh run`, and `gh api` for issues, PRs, CI runs, and advanced queries."
metadata:
  {
    "openclaw":
      {
        "emoji": "🐙",
        "requires": { "bins": ["gh"] },
        "install":
          [
            {
              "id": "brew",
              "kind": "brew",
              "formula": "gh",
              "bins": ["gh"],
              "label": "Install GitHub CLI (brew)",
            },
            {
              "id": "apt",
              "kind": "apt",
              "package": "gh",
              "bins": ["gh"],
              "label": "Install GitHub CLI (apt)",
            },
          ],
      },
  }
---

# GitHub Skill

Use the `gh` CLI to interact with GitHub. Always specify `--repo owner/repo` when not in a git directory, or use URLs directly.

## When to Use This Skill

**Use when:**

- Working with GitHub PRs (checks, reviews, merging)
- Managing GitHub issues (create, list, close)
- Checking CI/CD run status and logs
- Using `gh api` for advanced GitHub queries
- Creating releases or managing GitHub repos

**Don't use when:**

- Writing code locally → use `coding-agent`
- Managing project tasks (non-GitHub) → use `athena-tasks`
- Need email notifications about PRs → use `athena-email`
- Need security audit → use `healthcheck` or `clawdstrike`

**Success Criteria:**

- `gh` command returns expected output
- PR/issue created or updated correctly
- CI status accurately reported

### Common Pitfalls

**What NOT to do:**

- Running `gh` commands without `--repo owner/repo` when not in a git directory: fails silently or targets wrong repo
- Using `gh api` without `--jq` for filtering: returns massive JSON payloads
- Forgetting `--json` flag when you need structured output: returns human-readable but unparseable text
- Not checking `gh auth status` first: may fail with auth errors

**Alternative:**

- Always specify `--repo` when not in a project directory
- Use `--json field1,field2 --jq '.expression'` for clean output
- Run `gh auth status` at session start

## Pull Requests

Check CI status on a PR:

```bash
gh pr checks 55 --repo owner/repo
```

List recent workflow runs:

```bash
gh run list --repo owner/repo --limit 10
```

View a run and see which steps failed:

```bash
gh run view <run-id> --repo owner/repo
```

View logs for failed steps only:

```bash
gh run view <run-id> --repo owner/repo --log-failed
```

## API for Advanced Queries

The `gh api` command is useful for accessing data not available through other subcommands.

Get PR with specific fields:

```bash
gh api repos/owner/repo/pulls/55 --jq '.title, .state, .user.login'
```

## JSON Output

Most commands support `--json` for structured output. You can use `--jq` to filter:

```bash
gh issue list --repo owner/repo --json number,title --jq '.[] | "\(.number): \(.title)"'
```
