---
name: sociable-unit-tests
description: "Guide for writing sociable unit tests with directory isolation, no mocks, and transaction-level testing. Use when writing tests, reviewing test quality, or setting up test infrastructure for a Python project."
---

# Sociable Unit Tests

This skill covers three concerns:

- **Sociable Unit Testing Philosophy** — minimize mocking, test
  complete transactions through real code paths, isolate via
  environment variables and temp directories
- **Test Infrastructure and Documentation** — directory layout,
  test segregation, venv setup, and README documentation for
  running tests in Python repos
- **Agent Tests** — LLM inference tests with controlled context,
  no ambient configuration, predictable within model limits

The infrastructure and documentation section is general Python
project hygiene, not specific to sociable testing. It is included
here because the philosophy directly drives setup decisions (e.g.,
unit tests must be the default pytest target, integration suites
must be segregated by reliability profile). It may be separated
into a dedicated skill if skill composition conventions emerge.

---

## Part 1: Sociable Unit Testing Philosophy

### Core Principle

Write tests that verify complete features or transactions
end-to-end through your business logic layer — whether it's
called SDK, core, lib, domain, or service layer. Do NOT write
isolated "solitary" unit tests that mock internal collaborators.
Mocking hides integration bugs — the exact class of bugs tests
should catch.

**No mocks unless you ask.** Isolate via temp dirs and env vars.

### Testing Hierarchy

Bias strongly toward sociable unit tests. Cover everything
possible as fast, offline, deterministic unit tests before
reaching for any form of integration testing.

1. **Sociable unit tests** — the default. Every repo should
   have these. Cover all business logic through the core layer
   and interface layers using in-process testing (no network,
   no credentials, no external systems). This is where the
   vast majority of test value lives.

2. **Agent integration tests** (`tests/integration/agent/`) —
   add only when the repo's functionality involves LLM
   inference that needs behavioral verification. Keep them
   minimal, maximally reliable, and controlled. Do not look
   for opportunities to add agent tests to every project.

3. **Real-user integration tests** (`tests/integration/real-user/`)
   — last resort. Add only when the repo's core functionality
   depends largely or entirely on an external system that can
   only be tested with real, private user accounts that are
   not reusable across developers. Examples: proxying a
   third-party API with no sandbox environment, wrapping a
   platform where each developer must supply their own
   credentials and account data. Avoid entirely if unit tests
   with a local provider or contract testing can cover the
   same ground.

### What to Mock vs What NOT to Mock

**Mock only at system boundaries:**
- Network calls (APIs, HTTP, external services)
- System clocks (if time precision matters)
- Heavy external processes

**Never mock:**
- Internal helper functions or classes
- File system operations (use temp dirs instead)
- Configuration parsers (write real config files to temp dirs)
- Core layer methods called by interface layers (CLI, MCP, REST)

### Choosing a Boundary Strategy

When setting up tests for a project, there are two valid
approaches to handling external dependencies. Choose one and
use it consistently.

**Option A: Mock at the network call.** Stub or patch the HTTP
client so the core layer's own code runs fully but never hits
the wire. Simpler when the core layer is a thin wrapper around
an API.

**Option B: Local provider with real data.** Build an
alternative backend (e.g., an in-memory store, a local
database, a file-backed provider) that implements the same
interface as the real provider. Tests run against real data
through real code paths — no patching at all. This eliminates
the network boundary entirely rather than mocking it.

**When to prefer Option B:**
- The core layer has a provider/repository abstraction already
- Multiple test files need the same realistic data set
- You want to test filtering, sorting, and aggregation logic
  without maintaining mock response fixtures per test
- The project already uses this pattern — continue it

**If existing tests already follow one pattern, continue that
pattern.** Consistency within a repo matters more than
theoretical preference. Do not introduce Option A into a repo
using Option B or vice versa without explicit approval.

The local provider pattern is a form of what Martin Fowler calls
**narrow integration testing** — exercising the code that would
talk to an external service, but with the boundary replaced
entirely. The seed data that the local provider serves is
effectively an informal **contract** — a captured representation
of the external API's response shape. For repos that proxy
third-party platforms, consider periodically verifying that the
seed data still matches the real API shape (this is the core
idea behind **contract testing**, formalized by tools like Pact).

### Environment Variable Isolation

Every project that reads from config or data directories must
support env var overrides for those paths. Tests use these to
redirect to temp directories.

**Pattern (Python example):**

```python
# In the core layer (config.py or similar):
def get_config_dir() -> Path:
    path_str = os.environ.get("MY_APP_CONFIG_DIR")
    if path_str:
        return Path(path_str)
    return Path.home() / ".config" / "my-app"
```

```python
# In conftest.py:
@pytest.fixture(autouse=True)
def isolate_config(tmp_path, monkeypatch):
    """Auto-isolate every test from real config directories."""
    test_config = tmp_path / "config"
    test_config.mkdir(parents=True, exist_ok=True)
    monkeypatch.setenv("MY_APP_CONFIG_DIR", str(test_config))
```

Key points:
- `autouse=True` — applies to every test automatically
- `tmp_path` — unique per test, auto-cleaned
- `monkeypatch.setenv()` — auto-reverted when test ends, no
  cross-test leakage, no impact on parallel suites. Safer than
  raw `os.environ` manipulation.
- Place in `conftest.py` — central, no imports needed per test

**Isolate ALL writable paths:** config, data, cache, XDG
directories. If the core layer writes to it, tests must
redirect it.

### Local Provider Pattern (Option B)

When using a local provider for testing (Python example):

```python
# conftest.py
@pytest.fixture
def test_db_path(tmp_path):
    """Copy test data to temp directory."""
    src = Path(__file__).parent / "fixtures" / "test_data.json"
    dest = tmp_path / "test_data.json"
    shutil.copy(src, dest)
    return dest

@pytest.fixture
def local_provider(test_db_path):
    """Create a local provider with test data."""
    provider = LocalProvider(db_path=test_db_path)
    yield provider
    provider.close()
```

Seed data lives in `tests/unit/fixtures/` — either a static
file or generated from a seed file by a script. The seed file
is the only file that needs human review; generated data is
gitignored.

### Core-Layer-First Testing

All business logic lives in the core layer. CLI, MCP, REST,
and other interfaces are thin wrappers.

**Test the core layer directly** — don't test through interface
layers when verifying business logic.

```python
# Good: test core layer directly
def test_add_command_creates_config(tmp_path, monkeypatch):
    monkeypatch.setenv("MY_APP_CONFIG_DIR", str(tmp_path))
    result = core.add_command("fix", "Fix the bug")
    assert (tmp_path / "commands" / "fix.toml").exists()

# Bad: test through CLI subprocess
def test_add_command_cli(tmp_path):
    result = subprocess.run(["my-app", "cmds", "add", ...])
    # Slower, harder to isolate, tests CLI parsing not logic
```

### Interface Layer Tests

Thin, in-process tests for CLI, HTTP/REST, MCP, or other
interface layers. No out-of-process calls, no network traffic.
These verify wiring and request/response shape, not business
logic.

Examples:
- **HTTP/ASGI**: in-memory test client (e.g., httpx
  `ASGITransport` in Python, supertest in Node)
- **CLI**: built-in test runner (e.g., Click's `CliRunner` in
  Python, commander's programmatic API in Node)
- **MCP**: stdio protocol over pipes or in-process invocation

Interface tests belong in `tests/unit/` alongside core layer
tests — they're fast, deterministic, and need no credentials.

### Test Complete Transaction Flows

Test the full flow of a feature, not individual functions in
isolation.

Example: testing a "label email" feature:
1. Search for target email — confirm it exists without the label
2. Read email details (before) — assert label absent
3. Apply label — execute the operation
4. Read email details (after) — assert label present, other
   fields unchanged

This catches integration bugs between steps that isolated
function tests miss.

### Test Names

Test names describe **scenario + outcome**, not implementation
details.

Good:
- `test_not_installed_when_no_hooks_path`
- `test_refuses_to_overwrite_foreign_hook`
- `test_init_creates_manifest_and_dockerfile`
- `test_logs_food_to_current_date_directory`

Bad:
- `test_returns_false_when_file_missing`
- `test_respects_config`
- `test_returns_true`

---

## Part 2: Test Infrastructure and Documentation

### Directory Structure

```
tests/
  unit/                        # Fast, offline, no credentials (DEFAULT)
    conftest.py                # Shared fixtures (isolation, providers)
    fixtures/                  # Seed data, test configs
  integration/                 # Non-default suites (explicit invocation only)
    real-user/                 # Hits live third-party APIs with real
                               # credentials and real user data. Not stable
                               # across contributors — each needs their own
                               # account/credentials.
    agent/                     # LLM inference tests (see Part 3)
```

Most repos need only `tests/unit/`. Do not add integration
subdirectories speculatively — add them when a specific,
unavoidable testing need arises that unit tests cannot cover.
See the Testing Hierarchy in Part 1 for when each level is
appropriate.

### Test Segregation

Unit tests are the **only** default for bare `pytest`. Configure
this in `pyproject.toml`:

```toml
[tool.pytest.ini_options]
testpaths = ["tests"]
addopts = "--ignore=tests/integration"
```

Running tests:
```bash
pytest                                      # unit only (default)
pytest tests/integration/real-user/         # explicit suite
pytest tests/integration/agent/             # explicit suite
pytest tests/integration/                   # all integration
pytest tests/                               # everything
```

Passing a path as a positional arg overrides the configured
`testpaths` — no additional configuration or code needed.

### Unit Test Rules

- **Fast, local, no network, no credentials.**
- Subprocess calls only to ubiquitous tools (e.g., `git init`,
  `git commit` — never `git push`).
- Every test must be isolated from every other test and from
  the real filesystem.

### Virtual Environment and Dependencies

**Gitignore the venv directory.** Ensure `.venv/` is in
`.gitignore` before creating one.

**Test dependencies** go in the project's dev extras:

```toml
# pyproject.toml
[project.optional-dependencies]
dev = ["pytest", "pytest-asyncio"]
```

### Running Tests — README Template

The project's README (or CONTRIBUTING.md) must document how to
run each test suite. Template:

```markdown
## Testing

**Unit tests** run offline with no credentials:

    make setup       # bootstrap venv (once)
    pytest           # run unit tests

Or from a fresh clone:

    make test        # creates venv + runs unit tests

**Integration tests** require additional setup:

| Suite | Path | Prerequisites |
|-------|------|--------------|
| real-user | `tests/integration/real-user/` | Platform credentials (see Authentication) |
| agent | `tests/integration/agent/` | `ANTHROPIC_API_KEY` |

Run explicitly:

    pytest tests/integration/real-user/
    pytest tests/integration/agent/

These are excluded from the default `pytest` invocation.
```

### Makefile

`make test` always runs unit tests only. `make setup` is
independently callable so contributors can bootstrap the venv
and then use `pytest` directly for non-default suites:

```makefile
setup:
	python -m venv .venv
	.venv/bin/pip install -e ".[dev]"

test: setup
	.venv/bin/pytest
```

```bash
make test                                   # unit tests (fresh clone safe)
make setup && pytest tests/integration/...  # bootstrap then run any suite
```

### New Project Setup Checklist

1. Create `tests/unit/` with `conftest.py`
2. Decide on boundary strategy (Option A or B) — see Part 1
3. Configure `pyproject.toml` to ignore `tests/integration/`
   by default
4. Create isolation fixtures in `conftest.py`
5. Ensure all core layer config/data paths support env var
   overrides
6. Add `pytest` to dev dependencies
7. Gitignore `.venv/`
8. Add `setup` and `test` Makefile targets
9. Document test setup in README or CONTRIBUTING.md using the
   template above
10. Verify: bare `pytest` runs only unit tests

---

## Part 3: Agent Tests

Agent tests follow the sociable unit testing philosophy from
Part 1 — no mocks of internal collaborators, test complete
transactions, meaningful assertions — with one exception:
inference calls to an LLM are inherently non-deterministic, so
assertions check for reasonable signals rather than exact
outputs.

This section is optional. Most repos do not need agent tests.
Use this when a repo tests AI inference behavior and prompts
are expected to produce reliable, testable outputs.

### Requirements for Predictability

- **Known model version** — pin the model in the test, not in
  ambient config
- **Identical prompt and context each run** — no user-scoped,
  machine-scoped, or project context files that aren't fully
  controlled by the test
- **No tool calls to external services or user-specific data**
  — stub or mock all data backends with test fixtures
- **Assert on reasonable signals, not exact strings** — check
  that expected data appeared, a tool was called, or output
  contains key indicators

### Agent CLI Ambient Context

When testing via an agent CLI program (e.g., `claude -p`,
`gemini -p`) rather than a direct API call, these programs
load user-scoped and machine-scoped context by default —
global context files, plugin hooks, MCP registrations, shell
environment. Tests **must** disable all ambient context so the
only context the model sees is what the test explicitly
provides. For example, `claude --bare` strips all user and
project context.

### What Does NOT Belong in tests/integration/agent/

Tests where the model makes tool calls to services backed by
shifting data, user-specific accounts, or configurations that
vary across contributors. Those belong in
`tests/integration/real-user/` or are not suitable for
automated testing.

### Excluded from Default Test Runner

Agent tests require connectivity and API key env vars. They
are excluded from default `pytest` and documented in the
README alongside other integration suites.

```bash
pytest tests/integration/agent/
```

### Example Pattern

```python
# tests/integration/agent/conftest.py
import shutil, os, pytest

_HAS_KEY = bool(os.environ.get("ANTHROPIC_API_KEY"))
requires_api_key = pytest.mark.skipif(
    not _HAS_KEY, reason="ANTHROPIC_API_KEY not set"
)

@pytest.fixture
def mcp_config(tmp_path):
    """MCP config with stub backends only."""
    config = {"mcpServers": {
        "my-app": {"command": "my-app-mcp", "args": ["stdio"]},
        "stub-backend": {"command": "python", "args": ["tests/stubs/backend.py"]},
    }}
    path = tmp_path / "mcp.json"
    path.write_text(json.dumps(config))
    return str(path)
```

```python
# tests/integration/agent/test_prompts.py
@requires_api_key
def test_tool_invocation(mcp_config):
    result = subprocess.run(
        ["claude", "--bare", "--mcp-config", mcp_config,
         "-p", "List items with a balance over 500"],
        capture_output=True, text=True, timeout=120,
    )
    assert result.returncode == 0
    # Assert on signals, not exact output
    assert "500" in result.stdout or "balance" in result.stdout.lower()
```
