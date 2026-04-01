---
name: prioritize-github-issues
description: >-
  Scan GitHub issues across all repos and orgs the user belongs to, ranked by
  priority labels (P0-P9). Use when the user wants to see their issue backlog,
  triage priorities, or reprioritize issues across repos.
user-invocable: true
---

# Prioritize GitHub Issues

Scan open issues across the user's GitHub orgs and repos, rank them by priority,
and help the user reprioritize.

Requires the `gh` CLI to be installed and authenticated.

## Phase 1: Discover orgs and repos

List all orgs the user belongs to, plus their personal repos:

```bash
# User's orgs
gh api user/orgs --jq '.[].login'

# Repos for an org (public + private the user can see)
gh api orgs/<org>/repos --jq '.[].full_name' --paginate

# User's own repos
gh api user/repos --jq '.[].full_name' --paginate --affiliation owner
```

## Phase 2: Collect issues

For each repo, fetch open issues with their labels and creation date:

```bash
# All open issues for a repo with labels and dates
gh issue list --repo <owner>/<repo> --state open --json number,title,labels,createdAt,url --limit 200
```

Filter for issues that have priority labels matching the pattern `P0` through
`P9` (case-insensitive). Also collect issues with no priority label.

## Phase 3: Rank and display

Sort all collected issues by:

1. **Priority label** — P0 first (highest), P9 last. Issues with no priority
   label sort below P9.
2. **Within same priority** — newer issues (more recent `createdAt`) rank
   higher than older ones. The reasoning: newer issues reflect more current
   priorities; stale issues at the same level should sink.

Present as a ranked list grouped by priority level:

```
## P0 — Critical
1. owner/repo#123 — Issue title (2d ago)
2. owner/repo#456 — Issue title (5d ago)

## P1 — High
3. owner/repo#789 — Issue title (1d ago)
...

## Unprioritized
15. owner/repo#101 — Issue title (30d ago)
```

Include the repo name, issue number, title, age, and a clickable link.

## Phase 4: Scope refinement

If the list is long (more than ~20 issues), ask the user which orgs or repos
they want to focus on for this session. Filter the display accordingly. This
is a session preference — retain it for subsequent invocations in this
conversation if the agent supports that.

## Phase 5: Reprioritize

Ask the user if they want to reprioritize. If yes:

- Let them reassign priority labels to specific issues
- Apply changes via the `gh` CLI:

```bash
# Add a priority label
gh issue edit <number> --repo <owner>/<repo> --add-label "P1"

# Remove an old priority label
gh issue edit <number> --repo <owner>/<repo> --remove-label "P3"
```

- After changes, re-display the updated ranked list

## Label creation

If a repo doesn't have the priority labels yet, offer to create them:

```bash
# Create priority labels with a color gradient
gh label create "P0" --repo <owner>/<repo> --color "D73A4A" --description "Critical"
gh label create "P1" --repo <owner>/<repo> --color "E36209" --description "High"
gh label create "P2" --repo <owner>/<repo> --color "FBCA04" --description "Medium"
gh label create "P3" --repo <owner>/<repo> --color "0E8A16" --description "Low"
```

Extend through P9 if needed, but P0-P3 covers most use cases. Only create
labels the user agrees to — don't create all 10 by default.

## Notes

- The `gh` CLI must be authenticated with access to all relevant orgs
- Large orgs with many repos may take a moment to scan — show progress
- The agent should be creative in how it presents and helps manage the backlog
  beyond these hints. The commands above are reliable starting points, not
  rigid constraints.
