---
name: publish-skill
description: "Use after creating or updating a skill to publish it to a marketplace repo. Guides finding the skills source repo, validating the skill, placing it in the right collection, and committing/pushing."
metadata:
  version: "1"
---

# Publish a Skill to a Marketplace

## Prerequisites

The skill to publish must already exist somewhere locally (e.g.,
`~/.claude/skills/<name>/SKILL.md` or `~/.gemini/skills/<name>/SKILL.md`).

## Steps

### 1. Find the target marketplace repo

If `skills_marketplaces_list` is available, call it to discover
registered marketplace git URLs and aliases. If updating an existing
skill, `list_skills` or `get_skill` can show its `source` (marketplace
alias) and `source_path` (location within the repo). Work with the
user to confirm which marketplace to publish to if there are multiple.

If those tools are not available, check
`~/.config/ai-common/skills/source-repo.txt` for a saved local repo
path. If not found, ask the user and save the path there for future use.

Clone or locate the marketplace repo locally. If the user already has
it cloned, ask for the path rather than cloning again.

### 2. Validate the skill before publishing

Before copying to the source repo, verify:

- **Frontmatter is present and complete**: `name`, `description`, and
  `metadata.version` are required.
- **No references to specific consuming repos or projects.** Usage
  examples must be generic (e.g., `from my_app.sdk...`). If repo-specific
  references are found, rewrite them to be generic before publishing.
- **The skill name matches the directory name.**

### 3. Determine placement in the source repo

Glob the source repo for `*/SKILL.md` and `*/*/SKILL.md` to understand
how skills are organized into collections. Show the user the existing
structure and confirm which collection the skill belongs in.

If the skill already exists in the repo (same name), confirm with the
user that they want to update it.

### 4. Copy the skill

Copy the entire skill directory (not just SKILL.md — there may be
supporting files) into the source repo at the confirmed path.

### 5. Commit and push

- `git add` the skill directory
- `git diff --staged` to review
- Confirm with the user
- Commit with a descriptive message
- Push

## Tool Tiers

| Tier | Tool | Required |
|------|------|----------|
| 1 (zero-install) | git | Yes |
| 1b (widely available) | gh | No |
| 2 (author) | aicfg (for marketplace/manifest context) | No |
