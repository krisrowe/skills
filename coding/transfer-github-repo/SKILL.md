---
name: transfer-github-repo
description: >-
  Transfer a GitHub repository to a different owner/org with optional rename.
  Use when the user wants to move a repo between orgs or users, or rename a
  repo during transfer.
user-invocable: true
---

# Transfer GitHub Repository

Transfer a repo to a new owner (user or org) with an optional rename, then
update the local git remote to match.

## Steps

### 1. Confirm the source repo exists

```bash
gh repo view <owner>/<repo> --json url --jq '.url'
```

### 2. Transfer via API

The GitHub REST API for repo transfers requires a POST to
`/repos/{owner}/{repo}/transfer`. The `gh api` command may receive an
HTTP 307 redirect — this is normal and means the transfer was accepted.

```bash
gh api repos/<owner>/<repo>/transfer -X POST --input - <<'EOF'
{"new_owner":"<new-owner>","new_name":"<new-name>"}
EOF
```

If you get `HTTP 307 (Moved Permanently)`, the transfer is in progress.
Verify by checking the new location:

```bash
gh repo view <new-owner>/<new-name> --json url --jq '.url'
```

If the repo was already transferred once (redirect chain), use the numeric
repo ID instead:

```bash
# Get the repo ID first
gh api repos/<current-owner>/<current-name> --jq '.id'

# Transfer using the numeric ID (follows redirects)
gh api repositories/<repo-id>/transfer -X POST --input - <<'EOF'
{"new_owner":"<new-owner>","new_name":"<new-name>"}
EOF
```

### 3. Verify the transfer

```bash
gh repo view <new-owner>/<new-name> --json url --jq '.url'
```

GitHub automatically sets up redirects from the old URL to the new one.
Old clone URLs, links, and API calls will redirect.

### 4. Update local remote

If you have a local clone, update the remote:

```bash
cd /path/to/local/clone
git remote set-url origin https://github.com/<new-owner>/<new-name>.git
git remote -v  # verify
```

### 5. Update references

After transfer, search for old URLs in:
- README.md and other docs
- `setup.py` / `pyproject.toml` dependency URLs
- MCP server registrations
- CI/CD configs
- Other repos that reference the old URL (GitHub redirects handle git
  operations, but updating is good practice)

## Notes

- Transfers require admin access on the source repo and the ability to
  create repos in the target org.
- GitHub preserves all issues, PRs, stars, watchers, and git history.
- Old URLs redirect automatically — nothing breaks immediately, but
  update references over time.
- The `new_name` field is optional. Omit it to keep the current name.
