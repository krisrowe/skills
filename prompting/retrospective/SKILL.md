---
name: retrospective
description: >-
  Analyze a session's workflow for friction, errors, retries, and missed
  automation opportunities. Use when wrapping up a session, reviewing
  what went wrong, identifying churn, evaluating tool and skill
  effectiveness, or auditing agent autonomy. Triggers on: "retrospective",
  "retro", "what went wrong", "session review", "friction report",
  "what could be improved", "workflow analysis", "end of session review",
  "debrief", "post-mortem", "lessons learned", "what was unintuitive".
  Produces a structured report covering: timeline of errors and retries,
  skill and tool gaps, external knowledge dependencies, permission and
  access anomalies, and concrete improvement recommendations for skills,
  CLIs, APIs, and agent configuration.
---

# Session Retrospective

Analyze the current session's workflow to identify friction, wasted
effort, and opportunities for improved agent automation and autonomy.
Produce a structured report the user can act on — file issues, update
skills, fix tools, or adjust permissions.

## How to Run

Walk through the entire conversation from the beginning. For each
phase of work, answer the questions in each section below. Do not
skip sections — even a "nothing to report" is valuable signal.

## 1. Error and Retry Timeline

Reconstruct a chronological timeline of every failed attempt, error,
unexpected result, or retry in the session. For each incident:

- **What was attempted** — the specific tool call, command, or action
- **What went wrong** — the error message or unexpected behavior
- **Why it was attempted that way** — what guidance, assumption, or
  prior knowledge led to this approach (skill instructions, README,
  CLI help, guessing, prior session memory, framework source code)
- **What fixed it** — the workaround or resolution
- **How many round-trips were wasted** — count of tool calls or
  user interactions spent on this incident
- **Was the root cause the agent, the tool, the skill, or the
  environment?** — attribute clearly

Present as a numbered timeline, not grouped by category. Chronological
order reveals cascading failures and patterns that thematic grouping
hides.

## 2. Skill Effectiveness Audit

For every skill that was loaded or invoked during the session:

- **Was it invoked automatically or manually?** Did the agent
  recognize the need, or did the user have to type a slash command?
- **Did manual invocation unblock something?** Would the agent have
  been stuck without the user activating the skill?
- **Was the skill's guidance followed?** If not, what was ignored
  and why?
- **Were there errors or gaps in the skill's instructions?**
  Missing steps, incorrect examples, wrong defaults, outdated
  assumptions?
- **Did the skill contradict other sources?** (README, CLI help,
  other skills, actual tool behavior)
- **What would have saved time?** Specific additions, corrections,
  or restructuring that would have made the workflow more linear.

Also identify skills that were NOT invoked but should have been —
relevant skills that existed but were missed.

## 3. Tool and CLI Friction

For every CLI command, MCP tool, or API call that caused friction:

- **What was the tool/command?**
- **What was unintuitive about it?** (confusing output, misleading
  status, missing information, strict input requirements)
- **Did the agent have to guess at parameters?** (secret names,
  paths, URL formats, header requirements)
- **Was there adequate discoverability?** Could the agent have
  found the right invocation from help text, tool descriptions,
  or error messages alone?
- **What would the ideal behavior be?** Describe the fix.

Include MCP tool descriptions (`tools/list` schemas and docstrings)
in this analysis — were they sufficient for the agent to construct
correct calls on the first attempt?

## 4. Project Documentation and Configuration Gaps

Evaluate whether the current project's own documentation and
configuration provided adequate guidance for the workflow, or whether
missing or incomplete project-level information caused churn. This
applies to any project an agent works within — software repos,
document collections, data pipelines, operational runbooks, or any
workspace with context files and tooling.

### Context files (CLAUDE.md, GEMINI.md, CONTEXT.md, AGENT.md, README.md, CONTRIBUTING.md, etc.)

- **Was the agent able to complete the workflow using only the
  project's own docs?** If not, what was missing? (For code projects
  this means setup, build, test, deploy; for other projects it means
  whatever the workflow required — configuration, data access,
  operational steps, etc.)
- **Were there project-specific steps** (not covered by framework or
  tooling docs) that the agent had to discover by reading source,
  config, or runtime behavior, or by guessing or asking the user?
- **Did any project doc contradict actual behavior?** (e.g., docs
  say a path is `/mcp` but the service serves at `/`; docs say a
  default exists but the system requires explicit config)
- **Were prerequisites documented completely?** Environment
  variables, required secrets, credentials, access permissions,
  dependencies. Could a new user or agent complete the workflow
  from docs alone?
- **Were the right files preloaded as agent context?** Check which
  files are bootstrapped (e.g., `@README.md` in CLAUDE.md, context
  entries in `.gemini/settings.json`). Were there files with
  pertinent information that should have been preloaded but weren't?
  Were preloaded files missing information that the agent had to
  discover from code, config, or runtime behavior instead?
- **Did code or config contradict the preloaded context?** Flag
  cases where actual behavior (defaults, paths, endpoints, env var
  handling) differed from what the context files stated. These are
  high-priority fixes — preloaded context is what agents trust
  first, so stale or wrong context is worse than missing context.
- **Should information be reorganized across context files?** Is
  there setup, install, or deployment guidance buried in code
  comments, config files, or non-preloaded docs that belongs in a
  preloaded context file? Conversely, is there content in preloaded
  files that's stale, redundant with what the agent can discover
  from code, or irrelevant to the workflows agents actually
  perform?

### Project-owned commands and tool outputs

For commands and tools that belong to the project (not an external
framework) — CLIs the project ships, admin tools, scripts, custom
MCP tools, etc.:

- **Were error messages actionable?** Did they tell the user what
  to do, or just what failed?
- **Were success messages informative?** Did they confirm what
  happened and what to do next?
- **Were status/list commands complete?** Did they surface all the
  information needed for the next step, or did the agent have to
  make additional calls to fill gaps?
- **Were there missing commands?** Operations the workflow required
  that the project's CLI doesn't support (e.g., data import, bulk
  operations, historical backfill)?

### Attribution: project vs. framework

For each gap found above, clearly attribute whether the fix belongs
in the current project or in an upstream framework/tool the project
depends on. A project README can't fix a framework's misleading CLI
output, but it can document the workaround. A framework can't
document app-specific config, but it can provide better defaults or
discoverability.

## 5. External Knowledge Dependencies

Identify every point where the agent needed information from outside
the current project to proceed:

- **What information was needed?**
- **Where did it come from?** (framework source code, other repos,
  web search, user's memory, guessing)
- **Was access to that source explicitly approved by the user?**
  Or did it happen via permissive default settings?
- **Was this information critical?** Would the agent have been
  blocked or made wrong choices without it?
- **Could this information have been provided by a skill, README,
  CLI help, or tool description instead?** If so, where should it
  live?

This section catches cases where the agent "cheated" by reading
framework internals, peeking at other repos, or relying on knowledge
that a fresh user on a clean machine wouldn't have.

## 6. Permission and Access Anomalies

Review what files, directories, and systems the agent accessed:

- **Did the agent access files outside the current project?** List
  every external path read or written.
- **Was each external access explicitly approved by the user in this
  session?** Or was it auto-approved by permission settings?
- **Could the workflow be reproduced on a clean machine without
  those external accesses?** If not, what's missing?
- **How would the user test this workflow in a restricted
  environment?** What settings or flags would simulate a clean
  machine?

## 7. Data Flow and Migration Gaps

If the session involved data creation, migration, or import:

- **Were there operations the tools couldn't perform?** (e.g.,
  writing to historical dates, bulk import, cross-system migration)
- **What workarounds were used?** (direct storage access, scripts,
  manual steps)
- **Should the tool API support these operations?** Or is the
  workaround the intended path?

## 8. Ideal Linear Flow

Describe what the session's workflow SHOULD have looked like with
zero friction — the minimum sequence of commands, tool calls, and
user interactions to achieve the same outcome. Compare against what
actually happened. The delta between these two is the improvement
backlog.

## 9. Recommendations

Produce a prioritized table of concrete improvements, categorized by
what needs to change:

| Category | Target | Issue | Recommended Fix |
|----------|--------|-------|-----------------|
| Skill | skill-name | what's wrong | what to change |
| CLI | tool-name | what's wrong | what to change |
| MCP tool | tool-name | what's wrong | what to change |
| Docs | file/section | what's wrong | what to change |
| Config | setting | what's wrong | what to change |
| Framework | package | what's wrong | what to change |

For each recommendation, note whether it's:
- **Quick fix** — typo, missing example, wrong default
- **Enhancement** — new feature, new parameter, new tool
- **Design issue** — architectural change needed

## 10. Issue Candidates

From the recommendations, identify items worth tracking as GitHub
issues. For each, draft a one-line title and note which repo it
belongs to. Do not create the issues — present them for the user
to approve and prioritize.

## Output Format

Present the full report in the conversation. Use the section headers
above. Be specific — cite exact tool calls, error messages, file
paths, and line numbers where relevant. The report should be
actionable without re-reading the full session transcript.

After presenting the report, ask the user if they want to:
1. File any of the issue candidates
2. Update any skills with the recommended fixes
3. Save any findings to memory for future sessions
