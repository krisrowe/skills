# code-reuse

Not every duplicated pattern deserves a shared library. This skill gives your agent a decision framework for when to consolidate, when to copy, and when to leave it alone.

Covers the tradeoffs between managed snippet copies and shared packages, where to place shared code, and how to avoid premature abstractions that create more coupling than they solve.

Cross-platform — works with any AI agent that supports the SKILL.md format.

## Install

```bash
pipx install echomodel
em skills install code-reuse
```

Or install directly with Gemini CLI:

```bash
gemini skills install https://github.com/echo-skill/echoskill.git --path coding/code-reuse
```

