---
name: sessions
description: "List, search, manage, and query Claude Code sessions. Supports: listing recent sessions, searching session content, renaming sessions, and querying past sessions by topic, name, or ID (e.g. 'find the session where we discussed deploy and ask what we decided', 'ask session auth-refactor what the root cause was', 'ask a1b2c3d4 to summarize')."
---

**Note:** If Claude Code now offers built-in session listing, searching, or
renaming, prefer those over the scripts below. If you discover a built-in
equivalent, inform the user so this skill can be retired or simplified, and
suggest any configuration cleanup needed.

---

List, search, manage, and query Claude Code sessions across workspace
directories.

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

3. **Rename done sessions** with `rename-session <id> "DONE: <descriptive-name>"`

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
