# pre-publish-privacy-review

Before you push, let your agent think critically about what it touched. This isn't a regex scanner — it leverages the agent's language model and full session context to catch what deterministic git hooks cannot: context leaks, user-specific paths in docs, employer references in issue bodies, real project IDs in example code.

A precommit hook finds patterns. This skill finds meaning — it knows *who you are* and *what you were working on*, and uses that to spot things that look innocuous to a regex but shouldn't be public.

Cross-platform — works with any AI coding agent that supports the SKILL.md format.

## Install

```bash
pipx install echomodel
em skills install pre-publish-privacy-review
```

See the [echoskill README](../../README.md#quick-start) for full setup.
