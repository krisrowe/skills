---
name: agent-author
description: >-
  Guide for creating, configuring, and publishing agent and subagent
  definitions. Use when writing an agent .md file, setting up agent
  frontmatter, choosing where an agent belongs, configuring tools or
  model, or preparing an agent for distribution across projects or
  teams. Covers the consensus format shared by the major coding agent
  platforms that use markdown with YAML frontmatter.
---

# Agent Author

## Consensus format

Three major coding agent platforms share a common agent definition
format: **markdown files with YAML frontmatter**, where the frontmatter
configures the agent and the markdown body becomes the system prompt.

### Consensus players

- **Claude Code** ([docs](https://code.claude.com/docs/en/sub-agents))
- **Gemini CLI** ([docs](https://geminicli.com/docs/core/subagents/))
- **Cursor** ([docs](https://cursor.com/docs/context/subagents))

### Excluded platforms (too far from consensus)

- **OpenAI Codex** — uses `.toml` files, not markdown. Fields like
  `developer_instructions` replace the markdown body. Completely
  different file format.
  ([docs](https://developers.openai.com/codex/subagents))
- **Google Antigravity** — uses YAML files in `.antigravity/agents/`,
  not `.md` with frontmatter. Agent definition format is not well
  documented publicly.
- **Kiro (AWS)** — custom config format, agent definition spec not
  publicly documented at time of writing.

### No open standard

Unlike skills (where [agentskills.io](https://agentskills.io/specification)
provides a governing spec), there is **no open standard for agent
definitions**. The consensus format emerged independently across
platforms. The [AGENTS.md](https://agents.md/) standard (Linux
Foundation) covers project context files, not agent definitions.

## Directory layout consensus

All three consensus platforms follow the same two-tier pattern:

| Scope | Claude Code | Gemini CLI | Cursor |
|-------|-------------|------------|--------|
| Project | `.claude/agents/*.md` | `.gemini/agents/*.md` | `.cursor/agents/*.md` |
| User | `~/.claude/agents/*.md` | `~/.gemini/agents/*.md` | `~/.cursor/agents/*.md` |

The pattern is `.<tool>/agents/<name>.md` at both project and user
scope. Project-scope agents are specific to one codebase and can be
checked into version control. User-scope agents are personal and
available across all projects.

When agents share the same name across scopes, higher-priority
locations win (project overrides user in Claude Code and Gemini;
Cursor follows the same pattern).

## Common frontmatter fields

These fields are supported by all three consensus platforms:

| Field | Claude Code | Gemini CLI | Cursor | Notes |
|-------|-------------|------------|--------|-------|
| `name` | Required | Required | Required | Lowercase, hyphens. Identifier for the agent. |
| `description` | Required | Required | Required | When to delegate to this agent. All platforms use this for auto-delegation decisions. |
| `model` | Optional | Optional | Optional | Which model to use. All support `inherit` (use parent's model). |

The **markdown body** becomes the **system prompt** on all three
platforms.

## Platform-specific fields

Fields supported by only one or two platforms. Other platforms
silently ignore unknown frontmatter fields.

### Claude Code only

| Field | Purpose |
|-------|---------|
| `tools` | List of allowed tools (e.g., `Read`, `Bash(git *)`) |
| `disallowedTools` | Tools to deny from inherited set |
| `skills` | Preload skill content into agent context at startup |
| `permissionMode` | `default`, `auto`, `bypassPermissions`, etc. |
| `maxTurns` | Max agentic turns before stop |
| `mcpServers` | MCP servers available to this agent |
| `hooks` | Lifecycle hooks scoped to this agent |
| `memory` | Persistent memory: `user`, `project`, or `local` |
| `effort` | Reasoning effort: `low`, `medium`, `high`, `max` |
| `isolation` | `worktree` for git worktree isolation |
| `background` | Run as background task |
| `color` | Display color in UI |
| `initialPrompt` | Auto-submitted first user turn |

### Gemini CLI only

| Field | Purpose |
|-------|---------|
| `tools` | List of tool names, supports wildcards (`*`, `mcp_*`) |
| `kind` | `local` (default) or `remote` |
| `mcpServers` | Inline MCP server config isolated to agent |
| `temperature` | Model temperature (0.0–2.0) |
| `max_turns` | Max conversation turns (default 30) |
| `timeout_mins` | Max execution time in minutes (default 10) |

### Cursor only

| Field | Purpose |
|-------|---------|
| `readonly` | Restrict to read-only tools |
| `is_background` | Run as background task |

## Examples from each platform

### Claude Code

Source: [code.claude.com/docs/en/sub-agents](https://code.claude.com/docs/en/sub-agents)

```yaml
---
name: code-reviewer
description: Reviews code for quality, security, and best practices
model: sonnet
tools:
  - Read
  - Grep
  - Glob
  - Bash
skills:
  - api-conventions
  - error-handling-patterns
---

You are a senior code reviewer. Focus on:
1. Security vulnerabilities
2. Performance issues
3. Code style consistency
4. Error handling completeness

Provide specific, actionable feedback with file paths and line numbers.
```

### Gemini CLI

Source: [geminicli.com/docs/core/subagents/](https://geminicli.com/docs/core/subagents/)

```yaml
---
name: code-reviewer
description: Reviews code for quality, security, and best practices
model: inherit
tools:
  - read_file
  - search_code
max_turns: 15
timeout_mins: 5
---

You are a senior code reviewer. Focus on:
1. Security vulnerabilities
2. Performance issues
3. Code style consistency
4. Error handling completeness

Provide specific, actionable feedback with file paths and line numbers.
```

### Cursor

Source: [cursor.com/docs/context/subagents](https://cursor.com/docs/context/subagents)

```yaml
---
name: code-reviewer
description: Reviews code for quality, security, and best practices
model: inherit
readonly: true
---

You are a senior code reviewer. Focus on:
1. Security vulnerabilities
2. Performance issues
3. Code style consistency
4. Error handling completeness

Provide specific, actionable feedback with file paths and line numbers.
```

### Portable subset

An agent definition using only common fields works across all three:

```yaml
---
name: code-reviewer
description: Reviews code for quality, security, and best practices
model: inherit
---

You are a senior code reviewer. Focus on:
1. Security vulnerabilities
2. Performance issues
3. Code style consistency
4. Error handling completeness

Provide specific, actionable feedback with file paths and line numbers.
```

## Invoking agents from the CLI

### Claude Code (tested: v2.1.94)

Verified via `claude --help` and live test:

```bash
# Run a session as a named agent
claude --agent code-reviewer

# Run with a prompt (non-interactive)
claude --agent code-reviewer -p "review the last commit"

# List all configured agents
claude agents

# Invoke inline during a session via @-mention
@code-reviewer look at my recent changes

# Define an ephemeral agent inline (no file needed)
claude --agents '{"reviewer": {"description": "Reviews code", "prompt": "You are a code reviewer"}}'
```

The `--agent` flag replaces the default system prompt with the agent's
markdown body. CLAUDE.md and project memory still load normally.

**Tested:** Placed a portable agent at `~/.claude/agents/test-portable.md`
using only common fields (`name`, `description`, `model: inherit`).
`claude agents` listed it as `test-portable · inherit`.
`claude --agent test-portable -p "identify yourself" --max-turns 1`
responded correctly with the system prompt's instructions.

### Gemini CLI (tested: v0.36.0)

Verified via `gemini --help` and live test — **no `--agent` flag
exists**. Subagents are invoked via `@name` in prompts:

```bash
# Non-interactive with prompt
gemini -p "@code-reviewer review the auth module"

# Interactive session (agents auto-discovered)
gemini
# Then invoke inline:
@code-reviewer review the auth module
```

Gemini has no equivalent of Claude Code's `--agent` flag for running
a full session as a named agent.

**Tested:** Placed the same portable agent at
`~/.gemini/agents/test-portable.md`. `gemini agents list` discovered
it and listed it alongside built-in agents:
`test-portable: A test agent specifically for verifying cross-platform
agent formats.`
`gemini -p "@test-portable identify yourself"` delegated correctly:
`The test-portable agent identifies as a subagent for verifying
cross-platform agent format. It confirms that the portable agent
format is verified on the macOS platform.`
Same `.md` file, same frontmatter, works on both platforms.

### Cursor

Verified via [cursor.com/blog/cli](https://cursor.com/blog/cli) and
community docs — uses `agent chat` as the CLI entry point:

```bash
# Start an interactive agent session
agent chat "review the last commit"

# Non-interactive
agent chat -p "find bugs in src/"
```

Custom subagent selection from the CLI (equivalent to `--agent`) is
not yet supported as of 2026. Subagents are invoked via delegation
in the IDE or by the main agent based on the `description` field.
Not tested locally (cursor-agent CLI not installed).

## Writing agent definitions

### The `description` field drives delegation

All three platforms use `description` to decide when to delegate to
the agent. The same principles from skill descriptions apply:

- Use imperative framing: "Use when..." or "Reviews code when..."
- Cover synonyms for the task the agent handles
- Be specific enough to avoid false delegation
- Avoid disqualifying language that narrows selection

### The body is the system prompt

Keep it focused on *how the agent should behave*, not project context
(that's what CLAUDE.md / GEMINI.md / AGENTS.md files handle). The
body should define:

- The agent's role and expertise
- What to focus on and what to ignore
- Output format expectations
- Constraints and guardrails

### Consider skill preloading (Claude Code)

If the agent should always have certain knowledge loaded, use the
`skills:` frontmatter to preload skills. This is Claude Code only —
other platforms discover skills via progressive disclosure.

See the `skill-author` skill for guidance on when to preload vs.
let discovery handle it.

## Publishing agents

### Where agents live

| Destination | When to use |
|-------------|-------------|
| `.claude/agents/` (project) | Agent is specific to one codebase. Check into version control. |
| `~/.claude/agents/` (user) | Personal agent for all projects. |
| Plugin `agents/` directory | Distributed with a plugin. |
| Multiple `.<tool>/agents/` dirs | Cross-platform agent — copy the same .md to each tool's directory. |

### Cross-platform agents

To publish an agent that works across Claude Code, Gemini CLI, and
Cursor:

1. Write the agent using only common fields (`name`, `description`,
   `model`, and markdown body)
2. Copy the `.md` file to each platform's agents directory
3. Add platform-specific fields only where needed (e.g., `tools` for
   Claude, `timeout_mins` for Gemini)

Platform-specific fields are silently ignored by other platforms, so
a single file with mixed fields is safe — but keep the portable core
clean and add platform extensions deliberately.

## Verification sources

The following URLs were fetched and verified during development of
this skill:

- **Claude Code subagents**: https://code.claude.com/docs/en/sub-agents
  — frontmatter fields table, directory locations, skill preloading
- **Gemini CLI subagents**: https://geminicli.com/docs/core/subagents/
  — frontmatter fields table, directory locations, naming conventions
- **Cursor subagents**: https://cursor.com/docs/context/subagents
  — frontmatter fields, directory layout (confirmed via search results
  and community docs due to Vercel security on direct fetch)
- **Codex subagents**: https://developers.openai.com/codex/subagents
  — confirmed TOML format, excluded from consensus
- **AGENTS.md standard**: https://agents.md/
  — confirmed covers project context, not agent definitions
- **agentskills.io**: https://agentskills.io/specification
  — confirmed covers skills only, no agent definition spec
