# author-github-issue

Your agent files GitHub issues — but does it follow your team's conventions, avoid leaking PII, and write issues that someone else can actually pick up? This skill enforces structured authoring: temp file workflow, privacy rules, and self-contained issue bodies that don't assume conversation context.

Activates automatically whenever the agent creates, edits, or comments on a GitHub issue.

Cross-platform — works with any AI coding agent that supports the SKILL.md format.

## Install

```bash
pipx install echomodel
em skills install author-github-issue
```

Or install directly with Gemini CLI:

```bash
gemini skills install https://github.com/echo-skill/echoskill.git --path coding/author-github-issue
```

See the [echoskill README](../../README.md#quick-start) for full setup.
