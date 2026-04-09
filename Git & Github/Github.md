---
name: github
description: >
  Use this skill for ANY GitHub-related task: creating repos, pushing files,
  opening or reviewing pull requests, managing issues, creating branches,
  triggering or inspecting GitHub Actions/CI workflows, reading or summarizing
  codebases, forking repos, creating releases, and automating GitHub workflows
  via the REST API or gh CLI. Trigger immediately whenever the user mentions
  GitHub, a github.com URL, pull requests, issues, branches, commits, Actions,
  forks, releases, READMEs, or says anything like "put this on GitHub", "check
  my repo", "open a PR", "scaffold a project on GitHub", or "create a repo".
  Even vague requests should activate this skill — if GitHub is involved, this
  skill is involved.
---

# GitHub Skill

Authenticate once, then read repos, write code, manage PRs/issues/branches, and automate workflows — all from Claude.

---

## 1. Authentication & Tool Selection

Always check auth status first:

```bash
gh auth status 2>/dev/null && echo "GH_CLI_AVAILABLE" || echo "USE_API"
echo "TOKEN: ${GITHUB_TOKEN:0:8}..."
```

| Situation | Use |
|---|---|
| `gh` available + authed | `gh` CLI — simpler, handles pagination |
| `GITHUB_TOKEN` env set | REST API via `curl` or Python `requests` |
| Neither | Ask user to run `gh auth login` or provide a token |
| Read-only, public repo | `curl` unauthenticated (60 req/hr limit) |

**Never hardcode tokens.** Read from env: `$GITHUB_TOKEN` or `$(gh auth token)`.

---

## 2. Core Operations

### Create a repo + scaffold it

The flagship demo — from idea to live repo:

```bash
# Create repo
gh repo create <name> --public --description "<desc>" --clone
cd <name>

# Scaffold files
cat > README.md << 'EOF'
# <name>
<description>
EOF

git add . && git commit -m "feat: initial scaffold"
git push origin main

# Open in browser
gh repo view --web
```

When a user says "create a repo for my project idea," do this fully — create, scaffold a sensible file structure based on the project type, commit, push, and share the URL. Don't ask for permission at each step.

### Push files / commit code

```bash
# Single file via API (no clone needed)
gh api repos/{owner}/{repo}/contents/{path} \
  --method PUT \
  --field message="<commit msg>" \
  --field content="$(base64 < file.txt)"

# Multiple files — clone, edit, push
git clone $(gh repo view <repo> --json sshUrl -q .sshUrl)
cd <repo>
# make changes
git add . && git commit -m "<msg>" && git push
```

### Pull Requests

```bash
# Open a PR
gh pr create --title "<title>" --body "<body>" --base main --head <branch>

# List open PRs
gh pr list --repo <owner>/<repo>

# Read the diff
gh pr diff <number> --repo <owner>/<repo>

# Leave a review comment
gh pr review <number> --comment --body "<comment>"

# Merge
gh pr merge <number> --squash --delete-branch
```

When asked to "review a PR," actually read the diff and provide substantive feedback — don't just list the files changed.

### Issues

```bash
# Create
gh issue create --title "<title>" --body "<body>" --label bug,enhancement

# List
gh issue list --repo <owner>/<repo> --state open --limit 20

# Close with comment
gh issue close <number> --comment "Resolved in #<pr>"

# Triage
gh issue edit <number> --add-label "priority:high" --add-assignee <user>
```

### Branches

```bash
# Create and push
git checkout -b feature/<name>
git push -u origin feature/<name>

# Via API (no clone needed)
gh api repos/{owner}/{repo}/git/refs \
  --method POST \
  --field ref="refs/heads/<branch>" \
  --field sha="$(gh api repos/{owner}/{repo}/git/ref/heads/main --jq .object.sha)"
```

### GitHub Actions / CI

```bash
# Trigger a workflow
gh workflow run <workflow.yml> --ref main

# Check recent runs
gh run list --limit 10

# Watch live
gh run watch <run-id>

# Get logs
gh run view <run-id> --log

# Check latest commit status
gh run list --branch main --limit 1 --json status,conclusion,url
```

### Releases

```bash
# Create a release
gh release create v1.0.0 \
  --title "v1.0.0 - Initial Release" \
  --notes "First stable release" \
  --target main

# Upload assets
gh release upload v1.0.0 ./dist/app.tar.gz
```

---

## 3. Reading & Summarizing Repos

When a user shares a repo URL or asks to understand a codebase:

```bash
# Repo metadata
gh repo view <owner>/<repo> --json name,description,topics,stargazerCount,language

# File tree
gh api repos/{owner}/{repo}/git/trees/HEAD --field recursive=1 \
  --jq '[.tree[] | select(.type=="blob") | .path] | .[0:60]'

# Read a file
gh api repos/{owner}/{repo}/contents/{path} --jq '.content' | base64 -d

# Read README
gh api repos/{owner}/{repo}/readme --jq '.content' | base64 -d
```

**Summarization approach:**
1. Read the README first
2. Inspect the top-level file tree to understand structure
3. Read key files: `package.json` / `pyproject.toml` / `Cargo.toml`, main entrypoint, CI config
4. Synthesize: what it does, tech stack, how to run it, notable patterns

Be strategic — infer from structure, don't try to read every file.

---

## 4. REST API (when gh CLI unavailable)

Base URL: `https://api.github.com`

```bash
AUTH="Authorization: Bearer $GITHUB_TOKEN"
ACCEPT="Accept: application/vnd.github+json"

# GET
curl -s -H "$AUTH" -H "$ACCEPT" https://api.github.com/repos/{owner}/{repo}

# POST (create PR)
curl -s -X POST -H "$AUTH" -H "$ACCEPT" \
  -d '{"title":"Bug fix","body":"Fixes #42","head":"fix/bug","base":"main"}' \
  https://api.github.com/repos/{owner}/{repo}/pulls

# Check rate limit
curl -s -H "$AUTH" https://api.github.com/rate_limit | jq .rate
```

---

## 5. Common Flows

### "Put this on GitHub"
1. `gh repo create` with a name derived from the project
2. Clone, copy files in
3. Write a README if missing
4. Commit + push
5. Return the URL

### "Review my PR"
1. `gh pr diff <number>` — read the full diff
2. `gh pr view <number>` — read description and existing comments
3. Leave a structured review: summary → specific feedback → verdict

### "Why is CI failing?"
1. `gh run list --limit 5` — find the failing run
2. `gh run view <id> --log` — read logs
3. Identify root cause, suggest fix

### Scaffold by project type

| Type | Create |
|---|---|
| Node.js | `package.json`, `index.js`, `.gitignore`, `.github/workflows/ci.yml` |
| Python | `pyproject.toml`, `src/`, `tests/`, `.gitignore` |
| React | Vite scaffold, then push |
| CLI tool | `README.md`, `src/main.*`, `Makefile`, release workflow |

Always include `.gitignore` and a CI workflow appropriate to the stack.

---

## 6. Error Handling

| Error | Fix |
|---|---|
| `gh: command not found` | Fall back to REST API via `curl` |
| 401 Unauthorized | `gh auth login` or set `GITHUB_TOKEN` |
| 403 Forbidden | Check token scopes: needs `repo`, `workflow` |
| 404 Not Found | Verify repo exists and user has access |
| 422 Unprocessable | Missing required field in request body |
| Rate limited | Use authenticated requests; check `X-RateLimit-Remaining` |

Always surface the actual error with a clear explanation and fix.

---

## 7. Safety

- Never push secrets, tokens, or credentials to any repo
- Confirm before destructive operations: force-push, branch deletion, closing all issues
- Default to private repos unless user explicitly asks for public
- Don't `git push --force` without a warning
