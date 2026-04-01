# workspace-status

Quick health check across all your local repos. Your agent scans for uncommitted changes, unpushed commits, missing remotes, and dirty working trees — the stuff that quietly accumulates until something goes wrong.

Configurable via a roots file so it knows which directories to scan.

Cross-platform — works with any AI coding agent that supports the SKILL.md format.

## Install

```bash
pipx install echomodel
em skills install workspace-status
```

Or install directly with Gemini CLI:

```bash
gemini skills install https://github.com/echo-skill/echoskill.git --path coding/workspace-status
```

See the [echoskill README](../../README.md#quick-start) for full setup.
