# echoskill

Cross-platform AI agent skills for [Claude Code](https://code.claude.com/) and [Gemini CLI](https://geminicli.com/). Discover, share, and install reusable capabilities for your AI coding agents.

## Quick Start

### Claude Code

```bash
# Register this marketplace
em skills marketplace register echoskill https://github.com/echo-skill/echoskill.git

# Install a skill
em skills install capture-context

# List available skills
em skills list
```

### Gemini CLI

```bash
# Install all skills from a collection
gemini skills install https://github.com/echo-skill/echoskill.git --path coding

# Install one skill
gemini skills install https://github.com/echo-skill/echoskill.git --path prompting/nm
```

Gemini can install an entire collection at once — all skills under a folder are installed in one command.

### echomodel CLI (optional)

[em](https://echomodel.ai) provides cross-platform skill management for both Claude Code and Gemini CLI from a single tool. Not required — you can manage skills natively with each platform.

```bash
pipx install echomodel
em skills marketplace register echoskill https://github.com/echo-skill/echoskill.git
em skills install capture-context
```

---

## Collections

### [prompting/](prompting/)

User-invoked slash commands for prompt control.

| Skill | Description |
|-------|-------------|
| [nm](prompting/nm/SKILL.md) | Nevermind — discard the last prompt |
| [proceed](prompting/proceed/SKILL.md) | Continue after an interruption |
| [capture-context](prompting/capture-context/SKILL.md) | Capture all session context before ending a conversation |
| [pre-publish-privacy-review](prompting/pre-publish-privacy-review/SKILL.md) | Review content for privacy before publishing |

### [coding/](coding/)

Development workflow skills.

| Skill | Description |
|-------|-------------|
| [develop-unit-tests](coding/develop-unit-tests/SKILL.md) | Sociable unit testing with dir isolation and no mocks |
| [develop-skill](coding/develop-skill/SKILL.md) | Build portable cross-platform skills |
| [setup-agent-context](coding/setup-agent-context/SKILL.md) | Configure CLAUDE.md and .gemini/settings.json for a repo |
| [author-github-issue](coding/author-github-issue/SKILL.md) | Structured GitHub issue authoring with privacy rules |
| [publish-skill](coding/publish-skill/SKILL.md) | Publish skills to a marketplace |
| [code-reuse](coding/code-reuse/SKILL.md) | Find and reuse existing code patterns |
| [workspace-status](coding/workspace-status/SKILL.md) | Check workspace state across repos |
| [workstation-portability](coding/workstation-portability/SKILL.md) | Backup and portability for dotfiles, skills, and config |

### [claude/](claude/)

Claude Code-specific skills.

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

See [CONTRIBUTING.md](CONTRIBUTING.md) for details on adding skills.

## Links

- [echoskill.ai](https://echoskill.ai)
- Part of the [echomodel](https://echomodel.ai) ecosystem
