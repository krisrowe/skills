---
name: sessions
description: "List, search, manage, show, summarize, and query Claude Code sessions. Supports: listing recent sessions, searching session content, renaming sessions, getting session info by ID or name — full UUID, path, age, size, and ready-to-use resume/query commands (e.g. '/sessions info a1b2c3d4', 'session info auth-refactor'), summarizing a session (e.g. '/sessions summarize a1b2c3d4', 'summarize session auth-refactor'), and querying past sessions by topic, name, or ID (e.g. 'find the session where we discussed deploy and ask what we decided', 'ask session auth-refactor what the root cause was')."
---

**Note:** If Claude Code now offers built-in session listing, searching, or
renaming, prefer those over the scripts below. If you discover a built-in
equivalent, inform the user so this skill can be retired or simplified, and
suggest any configuration cleanup needed.

---

List, search, manage, and query Claude Code sessions across workspace
directories.

## Capabilities

| Invocation | Behavior |
|---|---|
| `/sessions` | List recent sessions (default 12) |
| `/sessions list 30` | List 30 most recent sessions |
| `/sessions find "deploy"` | Search full conversation content for keyword |
| `/sessions find "auth-fix" --name` | Find sessions by custom name (contains match) |
| `/sessions info a1b2c3d4` | Show full UUID, path, age, size, resume/query commands |
| `/sessions a1b2c3d4` | Same as info — bare ID triggers session info |
| `/sessions rename a1b2c3d4 "auth-fix"` | Rename a session |
| `/sessions tail a1b2c3d4` | Show last few messages from a session |
| `/sessions review` | Review open sessions, detect overlaps, rename done ones |
| `/sessions summarize a1b2c3d4` | Query the session with a summarize prompt (spawns claude) |
| `/sessions ask a1b2c3d4 "question"` | Query a past session non-interactively (spawns claude) |
| "find the session where we discussed X and ask..." | Search → confirm → query workflow |
| "ask session auth-refactor what the root cause was" | Resolve by name → query |
| "summarize session a1b2c3d4" | Equivalent to `/sessions summarize a1b2c3d4` |
| "which session made commit e2e4fb8" | Reverse lookup: grep-session or find-session by SHA |
| "grep session a1b2c3d4 for deploy" | Search within a specific session with context |

**Subagent filtering:** All commands hide subagent and test sessions by
default (sessions whose project path is outside `~`, e.g. `/var/folders/`
temp dirs). Use `--include-subagents` to show them.

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
~/.claude/skills/sessions/find-session "skill-review" --name  # match custom session name
```

Searches the full conversation content of recent sessions — not just summaries.
Use this when `--summary-contains` misses topics discussed mid-conversation.

Use `--name` to search custom session titles (set via `/rename`) instead of
full content. This is a case-insensitive contains match against the session name.

Options:
- `--most-recent=N`: how many recent sessions to search (default: 10)
- `--all`: require all search terms to match (default: any term matches)
- `--max-snippets=N`: max matching snippets per session (default: 3)
- `--name`: search custom session names instead of full content
- `--json`: output as JSON

## Renaming sessions

```bash
~/.claude/skills/sessions/rename-session <session-id-or-prefix> <new-name>
~/.claude/skills/sessions/rename-session a1b2c3d4 "auth-refactor" --status done
~/.claude/skills/sessions/rename-session a1b2c3d4 "old-experiment" --status archive
```

Writes the same `custom-title` entry that Claude Code's `/rename` does. Safe to
run multiple times — the last name wins.

Options:
- `--status STATUS`: prepend a status prefix to the name. Valid values:

| Status | Prefix | Meaning |
|--------|--------|---------|
| _(none)_ | _(no prefix)_ | No particular lifecycle status. |
| `done` | `DONE:` | Everything from this session is either fully completed in every detail, or all potentially relevant or valuable context has been fully captured for completion in a future session — e.g. in .md files, a TODO list, or a GitHub issue. If a `capture-context` skill is available, use it before marking done to ensure nothing is lost. |
| `open` | `OPEN:` | Intentionally flagged as incomplete before exit. The session has unfinished work that is NOT fully captured elsewhere. Resuming this session is expected. |
| `archive` | `ARCHIVE:` | Hidden from the default sessions list. Used for sessions that are no longer relevant but should not be deleted. `list-sessions` and `find-session` skip `ARCHIVE:` sessions by default (use `--include-archived` to show them). |

## Grep within a session

Search within a specific session by ID prefix or custom name. Returns
matching lines with context, the JSONL file path, and line numbers.

```bash
~/.claude/skills/sessions/grep-session <session-id-or-name> <pattern>
~/.claude/skills/sessions/grep-session a1b2c3d4 "deploy" -C 5
~/.claude/skills/sessions/grep-session a1b2c3d4 "deploy" --count
~/.claude/skills/sessions/grep-session auth-refactor "error" -i
```

Options:
- `-C N` / `--context N`: context lines around each match (default: 2)
- `--count`: only print match count and file path
- `-i` / `--ignore-case`: case-insensitive matching
- `--json`: output as JSON with file path, line numbers, and matches

Use this instead of `session-tail` when you need to understand what a
session actually did — tail only shows the ending, grep shows the work.

**Reverse lookup pattern:** To find which session made a commit:
```bash
~/.claude/skills/sessions/grep-session <session-id> "<commit-sha>"
```

## Session info

If the user invokes `/sessions <id>`, `/sessions info <id>`,
"session info a1b2c3d4", "session info auth-refactor", or provides
a session ID prefix or name as the only argument, get session info:

```bash
~/.claude/skills/sessions/session-info <session-id-or-prefix>
```

Display the output directly to the user.

Options:
- `--json`: output as JSON

The script resolves short prefixes (e.g. `a1b2c3d4`) to full UUIDs
and errors clearly on ambiguous prefixes.

Output includes:
- Full UUID, custom name (if renamed), project path
- Age, first user message, file size and line count
- Ready-to-use `cd && claude --resume` commands with real path and
  full UUID for resuming interactively or querying non-interactively

## Inspecting session endings

```bash
~/.claude/skills/sessions/session-tail <session-id-or-prefix>
```

Shows the last few conversational messages (human + assistant) from a session.
Use this to determine whether a session has incomplete work or ended cleanly.

Options:
- `--last=N`: number of messages to show (default: 3)
- `--role=human` or `--role=assistant`: filter to one role
- `--max-length=N`: max text per message (default: 500)
- `--json`: output as JSON

## Reviewing open sessions

When the user asks to review sessions, check for incomplete work, or find
overlapping sessions, follow this workflow:

1. **List recent sessions** with `list-sessions --json --count=30` (subagent
   and temp sessions are filtered out by default).

2. **Check each session's tail** with `session-tail <id> --last=2 --json`.
   Classify each as:
   - **Done**: last assistant message is a completed report, clean
     `/capture-context`, or answered question with no pending action
   - **Incomplete**: last assistant message says "now let me...",
     "next step...", gives instructions the user hasn't acted on, or was
     mid-edit/mid-action
   - **Blocked**: hit a permissions error or other blocker and stopped

3. **Rename done sessions** with `rename-session <id> "<descriptive-name>" --status done`

4. **Detect overlaps** between incomplete sessions:
   - Same repo + similar topic = likely overlap
   - Cross-repo but same feature area (e.g., a skill referenced in both repos)
   - Look at session names, summaries, and tail content for shared concepts

5. **Suggest sync prompts** for overlapping sessions. When session A has made
   progress that session B doesn't know about, suggest a prompt to send to
   session B:
   ```
   cd <path> && claude --resume <session-B-uuid> -p "Session <A-name> has \
   [describe changes/decisions]. Does this affect your current work? If so, \
   what needs to change?" --fork-session --max-turns 1
   ```

Present findings as a table with session ID, repo, name, status, and any
overlap notes.

## Session naming guidance

When wrapping up a session or when the user asks about session management, check
if the current session is unnamed or has a stale name. If so, suggest 2-3 short
names based on the work done and offer to rename using the script above.

Good session names are:
- Short (2-4 hyphenated words)
- Descriptive of the scope/outcome, not the first message
- Useful for finding the session later via `--summary-contains` or `find-session`

## Querying past sessions

This skill supports asking questions of past sessions. The user may say:

- "Find the session where we discussed deploy and ask what we decided"
- "Ask session auth-refactor what the root cause was"
- "Ask a1b2c3d4 to summarize the changes"
- "Summarize session a1b2c3d4" or "/sessions summarize a1b2c3d4"
- "What did we decide about naming in that session last week?"
- "Check if we discussed rate limiting in any recent session"

These fall into two patterns:

### Pattern A: User provides a session (by name or ID)

The user names a specific session by its 8-char ID prefix, full UUID,
or custom name. Resolve it:

- **By ID prefix**: run `find-session` or `list-sessions --json` and
  match the `session_id` field.
- **By name**: run `list-sessions --json` and match the `name` field.

Once resolved, skip to **Asking the question** below.

### Pattern B: User describes a topic

The user says "the session where we discussed X". You need to find it:

1. Extract search terms from the user's description.
2. Run:
   ```bash
   ~/.claude/skills/sessions/find-session "term1" "term2" --most-recent 20 --json
   ```
3. Present candidates to the user. For each, show:
   - Session ID (8-char prefix)
   - Age and project path
   - Name (if renamed) or first message summary
   - Why it seems relevant — quote a matching snippet
4. If multiple sessions match, rank by relevance and show the top 3-5.
5. **Ask the user to confirm** which session to query. Do not proceed
   without confirmation.

### Asking the question

Once you have a confirmed session, run:

```bash
cd <path> && claude --resume <full-uuid> -p "<question>" --fork-session --max-turns 1
```

**Requirements:**
- Use the **full UUID** from the JSON output, not the 8-char prefix.
  `claude --resume` with `-p` requires the full UUID.
- Always use `--fork-session` to avoid modifying the original session.
- Always use `--max-turns 1` unless the user explicitly wants more.
- `cd` to the session's project directory first (the `path` field from
  JSON output). File references in the conversation need to resolve.
- The `path` field uses `~` — expand it in the `cd` command.

### Presenting the answer

Show the response to the user. If they want to continue interactively,
suggest:

```
cd <path> && claude --resume <full-uuid>
```

### When no sessions match

Tell the user clearly and suggest:
- Broadening the search terms
- Increasing `--most-recent` (default is 20)
- Trying `list-sessions --summary-contains` for title/summary matches
