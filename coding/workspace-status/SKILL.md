---
name: workspace-status
description: "Check the health of local git repos: uncommitted changes, unpushed commits, missing remotes, dirty working trees. Use when asked about workspace status, repo hygiene, what needs pushing, or which projects have unsaved work."
---

# Workspace Status

Report the health of the user's local git repositories — uncommitted changes,
unpushed commits, missing remotes, and other at-risk state.

## Configuration

Settings are stored at `~/.config/ai-common/skills/workspace-status/roots.txt`.

### roots.txt format

One directory path per line. Blank lines and lines starting with `#` are ignored.

```
# Workspace roots to scan for git repos
~/projects
~/src
```

Paths may use `~` for the home directory. Expand `~` before use.

### First-run flow

1. Check if `roots.txt` exists and is non-empty.
2. If it exists, read it and proceed to discovery.
3. If it does not exist, ask the user:
   **"I don't have your workspace roots configured yet. Would you like me to
   try to locate your git repos automatically, or would you prefer to tell me
   where your projects live?"**
   - **Auto-detect**: Scan common locations (see platform table below) with
     `-maxdepth 2` and a short timeout. Present what was found and ask the
     user to confirm which roots to keep.
   - **Manual**: The user provides one or more paths.
4. Write the confirmed roots to `roots.txt`. Create the directory path if
   needed (`mkdir -p`).

### Updating roots

If the user says to add or remove a search root, update `roots.txt` directly.
Always show the user the updated file contents after a change.

## Discovery

Search each root in `roots.txt` for `.git` entries.

### Common root locations by platform (for auto-detect)

| Platform | Candidates to check |
|----------|-------------------|
| macOS | `~/Developer`, `~/Projects`, `~/projects`, `~/src`, `~/repos`, `~/code` |
| Linux (Debian, Ubuntu, Fedora, Arch) | `~/projects`, `~/src`, `~/repos`, `~/code`, `~/dev` |
| Crostini (ChromeOS) | `~/projects`, `~/src`, `~/repos` — home is `/home/<user>` |
| Windows | `%USERPROFILE%\projects`, `%USERPROFILE%\repos`, `%USERPROFILE%\source` |

During auto-detect, only check candidates that exist. For each that exists,
do a quick scan (`-maxdepth 2`) and report how many repos were found. Let the
user pick which roots to save.

**Do NOT search all of `~` (or `%USERPROFILE%`).** On most machines this
traverses massive irrelevant trees and can take minutes.

### Directories to skip

Exclude these from traversal — they are large, irrelevant, or produce false
positives:

**macOS only:**
- `~/Library`, `~/Music`, `~/Movies`, `~/Pictures`

**All Unix platforms (macOS, Linux, Crostini):**
- `node_modules`, `.venv`, `venv`, `vendor`, `__pycache__`, `site-packages`
- `.cache`, `.local`, `.npm`, `.cargo`, `.rustup`, `.gradle`, `.m2`
- `build`, `dist`, `target` (build output)
- Any directory starting with `.` directly under `~` (dotfile directories)

**Windows:**
- `AppData`, `node_modules`, `.venv`, `vendor`

### Find command (Unix)

```
find <ROOTS> -maxdepth 4 -name .git \
  -not -path '*/node_modules/*' \
  -not -path '*/vendor/*' \
  -not -path '*/.venv/*' \
  -not -path '*/venv/*' \
  -not -path '*/__pycache__/*' \
  2>/dev/null
```

- `-maxdepth 4` — repos deeper than this are typically vendored or nested deps
- `-name .git` without `-type d` — matches both directories (normal repos)
  and files (git worktrees)
- Redirect stderr to suppress permission errors
- Extract repo path by stripping `/.git` from each result

### Find command (Windows PowerShell)

```powershell
Get-ChildItem -Path <ROOTS> -Recurse -Filter .git -Depth 4 -Force -ErrorAction SilentlyContinue |
  Where-Object { $_.FullName -notmatch 'node_modules|vendor|\\.venv|venv|__pycache__' }
```

### Platform detection

- `uname -s` → `Darwin` (macOS), `Linux` (Linux/Crostini)
- Crostini specifically: check for `/mnt/chromeos` or `grep -qi cros /etc/os-release`
- Windows: PowerShell environment, or `$env:OS` equals `Windows_NT`

## Status checks

For each discovered repo, run these git commands:

1. **Uncommitted changes**: `git -C <repo> status --porcelain`
   - Non-empty output means dirty working tree
2. **Unpushed commits**: `git -C <repo> log --oneline @{upstream}..HEAD`
   - Non-empty output means local commits not yet pushed
   - This errors if the branch has no upstream — treat as "no remote tracking"
3. **Remote configured**: `git -C <repo> remote`
   - Empty output means no remote at all — highest risk, local-only work

On Windows, use the same git commands — git for Windows supports `-C`.

## Report format

### Repos with issues

```
## Workspace Status Report

### No remote configured (highest risk)
| Repo                   | Uncommitted | Unpushed | Notes           |
|------------------------|-------------|----------|-----------------|
| ~/projects/scratch-proj| 3 files     | —        | No offsite copy |

### Dirty or unpushed
| Repo                   | Uncommitted | Unpushed  | Branch  |
|------------------------|-------------|-----------|---------|
| ~/projects/my-api      | 5 files     | 2 commits | main    |
| ~/projects/my-app      | —           | 1 commit  | feature |

### Summary
3 of 22 repos need attention. 19 repos clean.
Scanned roots: ~/projects
```

### All clean

```
## Workspace Status Report

All 22 repos clean. Nothing to push or commit.
Scanned roots: ~/projects
```

### Column definitions

- **Repo**: home-relative path (`~/...` on Unix, `~\...` on Windows)
- **Uncommitted**: count of modified + added + deleted files, or `—` if clean
- **Unpushed**: count of commits ahead of upstream, or `—` if up to date
- **Branch**: current branch name (shown when there are unpushed commits)
- **Notes**: "No offsite copy" for repos with no remote; "No upstream" for
  branches with no tracking branch
