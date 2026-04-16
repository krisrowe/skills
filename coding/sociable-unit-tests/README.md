# sociable-unit-tests

Most unit test guides teach you to mock everything. This one teaches your agent to mock nothing.

Sociable unit tests verify complete transactions through your business logic — the same code paths your users hit in production. Isolation comes from temp directories and environment variables, not from replacing half your system with fakes.

The result: tests that actually catch integration bugs, the exact class of defects that mocked tests miss.

**What this skill covers:**
- When to mock (uncontrollable or slow) vs when not to (controllable and fast)
- Boundary strategies: mock at network call vs local provider with real data
- Transaction-level test design for core/business logic and interface layers
- Test infrastructure: directory layout, segregation, venv, Makefile, README templates
- Integration test hierarchy: unit (default) → agent (when needed) → real-user (last resort)
- Agent test patterns for LLM inference with controlled context
- Test naming conventions that describe behavior, not implementation

Cross-platform — works with any AI agent that supports the SKILL.md format.

## Install

```bash
pipx install echomodel
em skills install sociable-unit-tests
```

Or install directly with Gemini CLI:

```bash
gemini skills install https://github.com/echo-skill/echoskill.git --path coding/sociable-unit-tests
```

