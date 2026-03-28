---
name: develop-skill
description: "Guide for developing portable AI agent skills that work across Claude Code and Gemini CLI. Use when creating a new skill, reviewing a skill for portability, or registering a skill in a marketplace."
---

# Developing Portable Agent Skills

## Default Posture: Portability

Always aim for cross-platform skills unless there is a genuine reason a skill
can only work on one platform. Challenge assumptions about platform specificity:

- "Does this skill REALLY require Claude-only features, or can the instructions
  be written neutrally?"
- "Am I using platform-specific frontmatter out of habit or necessity?"

If the user says they're building for just one platform, evaluate whether the
skill's purpose is truly platform-specific by nature. Guide them toward portable
design when feasible.

## Skill Format: agentskills.io Standard

Skills use the [agentskills.io](https://agentskills.io/specification) open
standard, supported by 30+ tools (Claude Code, Gemini CLI, VS Code/Copilot,
Cursor, and many others).

A skill is a directory containing `SKILL.md` with YAML frontmatter:

```yaml
---
name: my-skill
description: When to invoke this skill and what it does
---

Instructions for the agent...
```

### Required Fields

- `name` — lowercase, hyphens, 1-64 chars, must match directory name
- `description` — tells the agent when to invoke. Be specific about triggers.

### Optional Platform Fields

These are Claude Code extensions that Gemini ignores:

- `disable-model-invocation: true` — slash-only, agent won't auto-invoke
- `user-invocable: false` — ambient-only, hidden from slash menu
- `allowed-tools: Bash, Read` — pre-approved tools
- `argument-hint: "[pattern]"` — autocomplete hint
- `context: fork` — run in isolated subagent
- `paths: "**/*.py"` — limit activation to matching files

### aicfg Extension Fields

These are stripped by aicfg during installation and never seen by any tool:

- `invocation: both | slash-only | ambient-only`
- `only: [claude]` — allowlist (mutually exclusive with exclude)
- `exclude: [gemini]` — denylist (mutually exclusive with only)
- `category: coding` — for listing and filtering

## No Scripts in Skills

**Skills are pure instructions.** Do not bundle executable scripts.

Why:
- Gemini blocks `run_shell_command` by default. Enabling it is a blanket
  permission for ALL shell commands, not scoped to the skill. Unacceptable.
- Claude's `${CLAUDE_SKILL_DIR}` scripting works but has no Gemini equivalent
  for standalone skills. Portability breaks.

### What Skills CAN Do

- Reference MCP tools by name: "call the `gapp_deploy` tool"
- Describe commands the agent should run: "run `git status` on each repo"
- Include hints, examples, or templates for the agent to adapt
- Trust the agent to formulate execution plans with user approval

### When a Script Is Required

If a skill absolutely requires deterministic code execution — a specific script
that must run exactly as written — it is a candidate for a **plugin** (Claude)
or **extension** (Gemini), not a standalone skill. That's what the packaging
layer is for: MCP servers, hooks, bundled scripts, and dependency management.

## Marketplace Structure

Skills live in git repos organized as directories:

```
skills-repo/
├── prompting/              ← collection
│   ├── nm/SKILL.md
│   └── proceed/SKILL.md
├── coding/                 ← collection
│   ├── develop-skill/SKILL.md
│   └── develop-unit-tests/SKILL.md
└── claude/                 ← platform-specific collection
    └── sessions/SKILL.md
```

**Collection** = a folder of skill subfolders. Installable as a group:
```bash
gemini skills install <url> --path coding
```

Or individually:
```bash
gemini skills install <url> --path coding/develop-skill
aicfg skills install develop-skill
```

## Installation

Skills are installed via `aicfg skills install` (cross-platform) or
`gemini skills install` (Gemini native). Claude has no skill CLI — aicfg
writes directly to `~/.claude/skills/`.

Both tools discover skills by scanning directories for `SKILL.md` files.
No registration or manifest needed beyond the file itself.

## Registering a New Skill

1. Create `<skill-name>/SKILL.md` in the appropriate collection folder
2. Include `name` and `description` in frontmatter (required)
3. Add `category` matching the collection name
4. Test locally:
   - Copy to `~/.claude/skills/<name>/` and verify Claude discovers it
   - Run `gemini skills install <local-path> --consent` and verify Gemini sees it
5. Commit and push to the marketplace repo
6. Verify: `aicfg skills list` shows it after marketplace update

## Writing Good Skill Instructions

- **Be specific about when to activate.** The `description` field is how agents
  decide to invoke. Include trigger phrases and scenarios.
- **Write for any agent.** Don't assume Claude or Gemini specific behaviors.
  Describe the desired outcome, not the tool-specific mechanism.
- **Reference tools by name** when the skill depends on them (e.g., MCP tools).
  Note that the tool must be installed separately.
- **Include examples** of commands, workflows, or outputs the agent should
  produce. Agents perform better with concrete examples.
- **Keep it focused.** One skill, one purpose. If it does two unrelated things,
  split it into two skills.
