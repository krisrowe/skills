# Skills Marketplace

Cross-platform AI agent skills for [Claude Code](https://code.claude.com/) and [Gemini CLI](https://geminicli.com/). Built on the [agentskills.io](https://agentskills.io/) open standard.

## Quick Start

### Gemini CLI

```bash
# Install all root-level skills from any git URL
gemini skills install https://github.com/YOUR_USERNAME/skills.git

# Install a collection
gemini skills install <git-url> --path coding

# Install one skill
gemini skills install <git-url> --path prompting/nm
```

Gemini can install an entire collection at once — all skills under a folder are installed in one command. Any git URL works (GitHub, GitLab, self-hosted, etc.).

### Claude Code (via [aicfg](https://github.com/krisrowe/aicfg))

```bash
# Register a marketplace (any git URL)
aicfg skills marketplace register my/skills <git-url>

# Install a skill (to both Claude and Gemini)
aicfg skills install nm

# Install to one platform only
aicfg skills install nm --target claude

# List all available skills
aicfg skills list
```

Claude Code has no native skill install CLI. [aicfg](https://github.com/krisrowe/aicfg) provides cross-platform skill management — same syntax for both tools.

---

## Collections

### [prompting/](prompting/)

User-invoked slash commands for prompt control.

| Skill | Description |
|-------|-------------|
| [nm](prompting/nm/SKILL.md) | Nevermind — discard the last prompt |
| [proceed](prompting/proceed/SKILL.md) | Continue after an interruption |

### [coding/](coding/)

Development workflow skills.

| Skill | Description |
|-------|-------------|
| [develop-unit-tests](coding/develop-unit-tests/SKILL.md) | Sociable unit testing with dir isolation and no mocks |
| [develop-skill](coding/develop-skill/SKILL.md) | How to build portable cross-platform skills |
| [setup-agent-context](coding/setup-agent-context/SKILL.md) | Configure CLAUDE.md and .gemini/settings.json for a repo |
| [workstation-portability](coding/workstation-portability/SKILL.md) | Backup and portability for dotfiles, skills, and config |

### [claude/](claude/)

Claude Code-specific skills that cannot be made platform-neutral.

| Skill | Description |
|-------|-------------|
| [sessions](claude/sessions/SKILL.md) | Search and manage Claude Code sessions |

---

## How It Works

Each skill is a directory containing a `SKILL.md` file with YAML frontmatter:

```yaml
---
name: my-skill
description: When to invoke this skill
---
Instructions for the agent...
```

Both Claude Code and Gemini CLI discover skills by scanning for `SKILL.md` files. No manifest or registration needed — the file IS the skill.

Claude-specific frontmatter (e.g., `disable-model-invocation`) is ignored by Gemini. See [CONTRIBUTING.md](CONTRIBUTING.md) for details on adding skills.
