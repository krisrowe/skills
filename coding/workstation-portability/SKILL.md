---
name: workstation-portability
description: "Workstation and workspace backup, portability, and reusability for code, configuration, AI tooling, dotfiles, plugins, and skills."
---

Help the user stay confident that their local workstation investment — configuration,
code, AI tooling — is portable and protected. This is not a migration checklist; it's
an ongoing posture of readiness.

## Philosophy

The goal is not "prepare to migrate" but "always be prepared." The user should feel
confident investing time in local configuration, custom skills, plugin setups, and
code without anxiety about loss. Every section below contributes to that confidence.

---

## 1. Dotfiles (`dotgit`)

The user manages dotfiles with `dotgit`, which supports multiple stores (e.g., default,
secrets, sensitive) backed by separate bare git repos. Use the MCP tools:

- `dot_status` — check each store for uncommitted changes
- `dot_list` — see what's tracked in each store
- `dot_sync` — commit and push to remote
- `dot_stores_list` — enumerate all stores

### Auditing across stores

To check all stores for unsaved changes:

```bash
for store in default secrets sensitive; do
  echo "=== $store ==="; dot status --store "$store"
done
```

Or use the MCP tools: call `dot_status` with each store name.

### Discovering untracked config directories

Compare what exists under `~/` against what's tracked:

```bash
# List dot-directories that exist on disk
ls -d ~/.*/ 2>/dev/null | sed "s|$HOME/||"

# Compare against tracked files across all stores
# (run dot_list for each store and combine)
```

Look for directories that:
- Are new (appeared since last audit)
- Contain configuration you'd want on another machine
- Are not ephemeral caches or runtime state

### Discovering new config directories (pending dotgit feature)

> **Status:** Not yet implemented in dotfiles-manager.
> Once implemented, update this section with the actual commands and remove this notice.

`dot status` currently only shows changes to already-tracked files (`status.showUntrackedFiles=no`).
A planned enhancement adds untracked file detection with cross-store awareness:

```bash
dot status                          # dirty + untracked by any store, respects audit-ignore
dot status -uno                     # dirty only (today's behavior)
dot status -uall                    # dirty + untracked (individual files in dirs)
dot status --isolated-store         # dirty + untracked by this store only, ignore other stores
dot status --isolated-store -uall   # same, individual files
```

An **audit-ignore file** at `~/.config/dotgit/audit-ignore` (gitignore syntax) will let
users suppress directories they've evaluated and don't want to track (e.g., `.cache/`,
`.npm/`). Managed via:

```bash
dot audit-ignore add <pattern>
dot audit-ignore list
dot audit-ignore remove <pattern>
```

The audit-ignore file is **intentionally local-only** — not tracked by any store. An
ignore list reveals what's installed on the machine, which could leak sensitive
information across store security boundaries. On a new machine, everything surfaces
as "unreviewed," which is correct — different machine, different evaluation.

**Until this is implemented**, the workaround is manual: compare `ls -d ~/.*/` against
`dot list` output across all stores to spot new, unevaluated directories.

### What typically belongs in dotgit

| Track | Skip |
|-------|------|
| `~/.config/<app>/settings*` | `~/.cache/` |
| `~/.claude/settings.json`, `skills/`, `CLAUDE.md` | `~/.npm/`, `~/.local/` |
| `~/.gemini/settings.json`, `commands/` | `~/.Trash/`, `~/.zsh_sessions/` |
| `~/.config/git/hooks/`, `~/.config/git/ignore` | Runtime state, OAuth tokens |
| `~/.ssh/config` (not keys — those go in secrets store) | History files, temp dirs |

---

## 2. Claude Code

### What to track (via dotgit)

- **`~/.claude/settings.json`** — plugin declarations, marketplace URLs, preferences
- **`~/.claude/skills/`** — custom skills (track the whole directory for future-proofing)
- **`~/.claude/CLAUDE.md`** — global instructions

### What NOT to track

`projects/`, `plugins/`, `sessions/`, `history.jsonl`, `tasks/`, `cache/`,
`backups/`, `plans/`, `telemetry/`, `shell-snapshots/`, `file-history/`

These are ephemeral, machine-specific, or regenerated automatically.

### Plugin restore on a new machine

Plugins are declared in `settings.json` under `enabledPlugins` and
`extraKnownMarketplaces`. The actual plugin files in `~/.claude/plugins/cache/`
are downloaded copies — no need to back them up.

To restore plugins on a new machine after syncing dotfiles:

```bash
# Register and update marketplaces declared in settings.json
claude plugin marketplace update <marketplace-name>

# Install each declared plugin
claude plugin install <plugin-name>@<marketplace-name> --scope user
```

SessionStart hooks in plugins handle dependency installation automatically
on first session.

### Session names

Sessions can be named with `/rename <name>` inside Claude Code. Names are stored
in the session JSONL as `{"type": "custom-title"}` entries. These do not port
across machines (sessions are local), but the `/sessions` skill can display them.

---

## 3. Gemini CLI

### What to track (via dotgit)

- **`~/.gemini/settings.json`** — MCP server registrations, auth type, feature flags.
  This is the equivalent of Claude's plugin declarations. Contains entries like:
  ```json
  {"mcpServers": {"tool-name": {"command": "...", "args": ["--stdio"]}}}
  ```
- **`~/.gemini/commands/`** — custom slash commands (`.toml` files)
- **`~/.gemini/GEMINI.md`** — if it's a symlink (e.g., to shared `~/.config/ai-common/CONTEXT.md`),
  track the symlink target

### What NOT to track

`oauth_creds.json`, `google_accounts.json`, `history/`, `tmp/`,
`installation_id`, `state.json`, `projects.json`, `trustedFolders.json`

These are machine-specific auth state, ephemeral data, or regenerated on login.

### Restore on a new machine

After syncing dotfiles, Gemini CLI will pick up `settings.json` and `commands/`
automatically. MCP servers referenced in `settings.json` must be installed
separately (e.g., via `pipx install <package>`). The `oauth_creds.json` is
regenerated on first `gemini` login.

---

## 4. Git Repos with Local Work

Repos with uncommitted changes or unpushed commits represent work at risk.
To discover them under a given root:

```bash
# Find repos with uncommitted or unpushed work
find <ROOT> -maxdepth 4 -name .git -type d 2>/dev/null | while read gitdir; do
  repo=$(dirname "$gitdir")
  status=$(git -C "$repo" status --porcelain 2>/dev/null)
  unpushed=$(git -C "$repo" log --oneline @{upstream}..HEAD 2>/dev/null)
  no_remote=$(git -C "$repo" remote 2>/dev/null | wc -l | tr -d ' ')
  if [ -n "$status" ] || [ -n "$unpushed" ] || [ "$no_remote" = "0" ]; then
    echo ""
    echo "=== $repo ==="
    [ -n "$status" ] && echo "  Uncommitted changes"
    [ -n "$unpushed" ] && echo "  Unpushed commits: $(echo "$unpushed" | wc -l | tr -d ' ')"
    [ "$no_remote" = "0" ] && echo "  WARNING: No remote configured"
  fi
done
```

Replace `<ROOT>` with the workspace directory (e.g., the user's projects root or `$HOME`).

Repos with no remote are the highest risk — local-only work with no offsite copy.

---

## 5. Confidence Check

A periodic (or on-demand) sequence to verify readiness:

### Quick check

1. **Dotfiles synced?** — `dot_status` across all stores. Any uncommitted changes?
2. **Repos clean?** — Run the git scan from Section 4 against the workspace root.
3. **Plugin declarations current?** — Compare `~/.claude/settings.json` `enabledPlugins`
   against what's actually installed (`~/.claude/plugins/installed_plugins.json`).

### Deeper audit (monthly or after installing new tools)

4. **New dot-directories?** — `ls -d ~/.*/ | sed "s|$HOME/||"` and compare against
   what's tracked across stores. Evaluate anything new.
5. **Gemini MCP servers match Claude MCP?** — Cross-check that tools available in
   one AI assistant are registered in the other if desired.
6. **SSH keys backed up?** — Verify `~/.ssh/config` is tracked (secrets store) and
   keys are in a secure backup.

### What "ready" looks like

- All `dot_status` stores show clean (no uncommitted changes)
- All `dot_sync` stores have a remote and are pushed
- No git repos under the workspace root have uncommitted changes or missing remotes
- `settings.json` for both Claude and Gemini are tracked and current
- Custom skills and commands are tracked in dotgit
