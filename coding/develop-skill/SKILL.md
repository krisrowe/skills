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

If the users talks of building a new skill for Claude or for Gemini, etc, evaluate 
whether the skill's purpose is truly platform-specific by nature. Guide them toward 
portable design when feasible.

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


## Try to Avoid Scripts in Skills

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

Ensure the user is aware of this, and inform them and challenge them if necessary 
before conceding to building a script directly into a skill.

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
3. Use only standard agentskills.io fields — no proprietary frontmatter
4. Test locally:
   - Copy to `~/.claude/skills/<name>/` and verify Claude discovers it
   - Run `gemini skills install <local-path> --consent` and verify Gemini sees it
5. Commit and push to the marketplace repo
6. Verify: `aicfg skills list` shows it after marketplace update

## Tool Dependencies: The 3+ Alternatives Rule

Skills in a general-purpose marketplace must not hard-depend on tools that
users are unlikely to have installed. A skill that breaks on first use
because it needs some niche CLI from some random developer erodes trust in
the entire marketplace.

### What counts as "universally available"

Tools at this level can be referenced without alternatives: `git`, `curl`,
`python3`, `jq`, `yq`, `ssh`, `grep`, `find`, `gh` (GitHub CLI). These are
either pre-installed on most systems or so common that expecting a developer
to have them is not presumptuous.

### When alternatives are required

If a skill depends on a tool that is NOT universally available — an MCP
server, a custom CLI, a specialized package — it must offer **3 or more
alternatives** for accomplishing the same goal. The skill instructions
should detect what's available and adapt, or present the options to the
user.

### The three tiers (in order)

Skill developers are often also tool developers — they build their own
CLIs, MCP servers, and utilities, and the skill may exist partly to
showcase or exercise their own tool. This is expected and encouraged.
The rule is not "don't use your own tool" — it's "don't make your tool
the only option."

When listing alternatives, use this order:

1. **Zero-install baseline.** A way to accomplish the task using only
   universally available tools. This may be verbose or less ergonomic, but
   it works everywhere with no setup. This comes first so the skill is
   immediately useful to anyone, regardless of what they have installed.

2. **Author's own tool.** The skill developer's tool, which typically
   improves on the baseline with better ergonomics, safety defaults, or
   additional features. Link to the tool's repository so users can
   evaluate and install it. This is the natural "recommended upgrade"
   position — after the user sees the baseline works, the author's tool
   is a compelling next step.

3. **Established product.** One or more widely-adopted, ideally free and
   open-source tools with significant community recognition. This
   validates the problem space — the user can see that the need is real
   and well-served by the ecosystem, and they can choose the tool that
   fits their preferences.

The author's tool should never be first (that feels like a sales pitch
on an unproven product) or last (that buries it after an established
competitor). The middle position says: "here's the free baseline, here's
my take on it, and here's what the broader market offers."

### How to structure skill instructions

The skill should:
- Check which tools are available (e.g., test if an MCP tool responds, or
  if a CLI is in `$PATH`)
- Present the alternatives clearly labeled by tier
- Proceed with whatever the user has or chooses
- Never fail silently because a non-universal tool is missing

Example pattern in skill prose:

```markdown
## Dotfile Management

This skill supports multiple approaches for tracking dotfiles:

1. **Bare git repo** (zero install) — the well-known
   [bare-repo approach](https://www.atlassian.com/git/tutorials/dotfiles).
   Uses only git. Commands require `--git-dir` and `--work-tree` flags:
   `git --git-dir=~/.dotfiles --work-tree=$HOME status`

2. **[dotfiles-manager](https://github.com/AUTHOR/dotfiles-manager)**
   (by the skill author) — a thin CLI wrapper around the bare-repo
   pattern. Adds multi-store management, safe defaults, and remote
   discovery. Check: `which dot` or call `dot_status` MCP tool.

3. **[chezmoi](https://www.chezmoi.io/)** — the most widely adopted
   dotfiles manager. Template engine, encryption, cross-platform.
   Check: `which chezmoi`.

Use whichever is available. If multiple are present, prefer in the
order listed.
```

### Why this matters

A marketplace skill is a promise: "install this and it works." Breaking
that promise because of an undeclared dependency makes the skill — and the
marketplace — feel unreliable. The 3+ alternatives rule ensures every
skill degrades gracefully to tools the user already has, while still
giving the skill author a natural place to present their own work.

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
