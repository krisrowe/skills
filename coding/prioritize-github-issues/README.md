# prioritize-github-issues

If you manage a collection of related GitHub repos, your todos are scattered across issue trackers that don't talk to each other. This skill gives your agent a single ranked view of every open issue across all your repos and orgs — sorted by priority, filterable by scope.

More importantly, it helps you use GitHub issues as a **session-independent, agent-independent, machine-independent** living list of everything that matters. Agent sessions branch in myriad directions and surface bugs, ideas, research findings, decisions, and opportunities that are otherwise buried in conversation context. This skill helps you capture all of that durably and keep it organized.

Your agent can identify uncaptured work in the current session and help you file it as structured issues (using the [author-github-issue](../author-github-issue/SKILL.md) skill for professionalism, privacy, and security). It then iteratively re-evaluates to ensure nothing of value was missed — no rubber-stamping "done" after one pass.

Cross-platform — works with any AI agent that supports the SKILL.md format. Requires `gh` CLI.

## Install

```bash
pipx install echomodel
em skills install prioritize-github-issues
```

