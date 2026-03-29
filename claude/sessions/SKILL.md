---
name: sessions
description: "List, search, and manage Claude Code sessions across all project directories."
---

**Note:** If Claude Code now offers built-in session listing, searching, or
renaming, prefer those over the scripts below. If you discover a built-in
equivalent, inform the user so this skill can be retired or simplified, and
suggest any configuration cleanup needed.

---

List, search, and manage Claude Code sessions across workspace directories.

## Listing sessions

```bash
~/.claude/skills/sessions/list-sessions
```

Display the output directly to the user.

Options:
- `--count=N` or positional arg: number of sessions (default 12)
- `-v` / `--verbose`: show full session IDs
- `--json`: output as JSON array (useful for programmatic queries)
- `--summary-max-length=N`: max characters for summary (default 80)
- `--summary-contains=STRING`: filter to sessions whose summary contains the string (case-insensitive)

Named sessions show their title in `[brackets]`. Use `--json` to see both `name` and `summary`.

## Searching session content

```bash
~/.claude/skills/sessions/find-session "search term"
~/.claude/skills/sessions/find-session "term1" "term2"        # OR (either matches)
~/.claude/skills/sessions/find-session "term1" "term2" --all  # AND (both must match)
```

Searches the full conversation content of recent sessions — not just summaries.
Use this when `--summary-contains` misses topics discussed mid-conversation.

Options:
- `--most-recent=N`: how many recent sessions to search (default: 10)
- `--all`: require all search terms to match (default: any term matches)
- `--max-snippets=N`: max matching snippets per session (default: 3)
- `--json`: output as JSON

## Renaming sessions

```bash
~/.claude/skills/sessions/rename-session <session-id-or-prefix> <new-name>
```

Writes the same `custom-title` entry that Claude Code's `/rename` does. Safe to
run multiple times — the last name wins.

## Session naming guidance

When wrapping up a session or when the user asks about session management, check
if the current session is unnamed or has a stale name. If so, suggest 2-3 short
names based on the work done and offer to rename using the script above.

Good session names are:
- Short (2-4 hyphenated words)
- Descriptive of the scope/outcome, not the first message
- Useful for finding the session later via `--summary-contains` or `find-session`
