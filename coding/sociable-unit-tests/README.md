# sociable-unit-tests

Most unit test guides teach you to mock everything. This one teaches your agent to mock nothing.

Sociable unit tests verify complete transactions through your business logic — the same code paths your users hit in production. Isolation comes from temp directories and environment variables, not from replacing half your system with fakes.

The result: tests that actually catch integration bugs, the exact class of defects that mocked tests miss.

**What this skill covers:**
- When to mock (system boundaries only) vs when not to (everything else)
- Directory isolation patterns for stateful operations
- Transaction-level test design for your core/lib layer and optionally at programmable interface layers (CLI, MCP, API)
- Test naming conventions that describe behavior, not implementation

Cross-platform — works with any AI coding agent that supports the SKILL.md format.

## Install

```bash
pipx install echomodel
em skills install sociable-unit-tests
```

Or install directly with Gemini CLI:

```bash
gemini skills install https://github.com/echo-skill/echoskill.git --path coding/sociable-unit-tests
```

See the [echoskill README](../../README.md#quick-start) for full setup.
