# transfer-github-repo

Move a GitHub repo between orgs or users with an optional rename in a single operation. Your agent handles the API call, follows redirects, verifies the transfer, and updates your local remote.

Includes the workaround for GitHub's redirect chain when a repo has been transferred before.

Cross-platform — works with any AI coding agent that supports the SKILL.md format.

## Install

```bash
pipx install echomodel
em skills install transfer-github-repo
```

Or install directly with Gemini CLI:

```bash
gemini skills install https://github.com/echo-skill/echoskill.git --path coding/transfer-github-repo
```

See the [echoskill README](../../README.md#quick-start) for full setup.
