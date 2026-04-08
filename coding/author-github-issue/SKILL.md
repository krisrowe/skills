---
name: author-github-issue
description: >-
  Use when creating (gh issue create), editing (gh issue edit), or
  commenting on (gh issue comment) a GitHub issue — whether the user
  asks directly or the agent decides one should be filed for a bug,
  feature, design decision, or any other trackable item.
---

# Authoring GitHub Issues

## Workflow: File-Based Issue Creation and Editing

Always use a temporary `.md` file and `--body-file` for issue operations.
Never pass issue body content inline via `--body` or heredocs. This ensures:

- The user sees the full content via the file-editing tool before it ships.
- Edits are visible as diffs, not opaque string replacements.
- No content is lost to shell escaping or truncation.

### Creating a New Issue

1. Write the issue body to a temp file:
   ```
   Write tool → /tmp/issue-<slug>.md
   ```
2. Let the user review/approve the file content.
3. Create the issue:
   ```bash
   gh issue create --repo <owner>/<repo> --title "<title>" --body-file /tmp/issue-<slug>.md
   ```

### Updating an Existing Issue

1. Fetch the current body to a file:
   ```bash
   gh issue view <number> --repo <owner>/<repo> --json body --jq '.body' > /tmp/issue-<number>.md
   ```
2. Edit the file using the file-editing tool (Edit on Claude Code,
   `edit_file` on Gemini CLI). This shows the user a diff of what changed
   rather than replacing the entire issue body opaquely.
3. Update the issue from the file:
   ```bash
   gh issue edit <number> --repo <owner>/<repo> --body-file /tmp/issue-<number>.md
   ```

### Why This Matters

- **Transparency**: The user sees exactly what will be published before it ships.
- **Diff visibility**: File edits show before/after in the agent's native tool
  approval UI, so the user can review deltas naturally.
- **No data loss**: Large issue bodies don't get truncated by shell quoting,
  heredoc limits, or argument length limits.
- **Auditability**: The temp file persists for the session if the user wants
  to review what was sent.

## Content Rules for GitHub Issues

**Treat every issue as a public-facing artifact.** Even on private repos,
issues may become public (collaborators, open-sourcing, forks). Write from
the perspective of someone contributing to a reusable product, not from the
perspective of the current user's personal workflow.

### Never Include User-Specific Information

- PII: names, emails, usernames, employer names, phone numbers
- User workstation-specific paths (home directories, workspace roots,
  local directory layouts) or configurations
- Cloud/service identifiers: GCP project IDs, API keys, document IDs,
  account IDs
- Examples drawn from the user's local data, use cases, or environment
- Anything that reveals the author's identity as a user of the repo
- References to the user's other projects, repos, or GitHub repositories —
  unless the repo this issue belongs to directly imports or depends on that
  other project (e.g., it's declared as a dependency in package config).
  A personal project that *uses* this repo is not a valid reference;
  only direct code-level bindings between repos justify cross-references.

### Instead

- Use generic placeholders: `~/projects/my-app`, `your-project-id`,
  `user@example.com`
- Describe needs from the repo's perspective, not the user's workflow
- Write issue titles and bodies that would make sense to any contributor
- Focus on how the change benefits the product broadly

### Issue Body Structure

- **Lead with the problem.** What's broken, missing, or awkward?
- **Propose a solution.** Be specific about the desired behavior.
- **Include context for decisions.** Why this approach over alternatives?
- **Add a work breakdown** with checkboxes for implementation steps.
- **Update the body as scope evolves** — don't add comments for scope
  changes. The body should always reflect the current understanding.
  Comments are for discussion, not revisions to the spec.

### Call for documentation of rationale and principles

When an issue introduces design principles, architectural decisions,
or rationale that should guide future work, the issue body should:

1. **Capture the principles in the issue itself** so they're visible
   during implementation and review.
2. **Call for those principles to be documented in repo context files**
   as part of the implementation work breakdown. This means adding
   them to `CONTRIBUTING.md` (for architecture, design constraints,
   and conventions) or `README.md` (for user-facing behavior and
   project identity) — whichever is appropriate per the repo's
   context file conventions.

Why this matters: `CONTRIBUTING.md` and `README.md` are loaded as
agent context at session start (via `CLAUDE.md` `@` imports and
`.gemini/settings.json` context declarations — see the
`setup-agent-context` skill). Principles documented there
automatically inform every future agent session working on the repo.
Principles captured only in an issue body are invisible to agents
unless the unlikely event happens where someone goes back later
while maintaining or extending the project and reads an old,
hopefully closed issue.

The work breakdown should include explicit items like:
- `[ ] Document [principle/rationale] in CONTRIBUTING.md`
- `[ ] Update README.md if [decision] affects user-facing behavior`

This ensures that closing the issue means the rationale is durably
captured where agents and contributors will see it — not buried in
a closed issue that no one reads again.
