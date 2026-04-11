---
name: author-mcp-app
description: "Build, structure, migrate, or review Python MCP servers and web APIs. Use when asked to create a new MCP server, structure a solution repo, add multi-user auth, set up a data store, migrate an existing app, review an app against standards, or any question about building a deployable Python MCP service — \"create an MCP server\", \"add auth to my app\", \"how should I structure this\", \"set up user management\", \"make this multi-user\", \"review my solution\", \"is this ready to deploy\", etc."
disable-model-invocation: false
user-invocable: true
---

# Author MCP App

## Overview

This skill guides users through building Python MCP servers and
web APIs that are self-contained, deployable apps. Solutions built
with this guidance work locally (stdio, single user) and deployed
(HTTP, multi-user) without code changes.

## Framework Choice

This skill supports two approaches:

### Recommended: mcp-app (config-driven)

[mcp-app](https://github.com/echomodel/mcp-app) is a config-driven
framework that wraps FastMCP and Starlette. You define tools as
plain async functions, configure via `mcp-app.yaml`, and run with
two commands. No framework imports in your tool code.

**What the user gets with mcp-app:**

- **Zero framework code in tools.** Tools are plain async functions.
  No decorators, no FastMCP imports, no framework coupling in
  business code. mcp-app discovers public async functions from the
  module declared in `tools:` and registers them automatically —
  function names become tool names, docstrings become descriptions,
  type hints become schemas.

- **User identity and profile.** Identity middleware runs by default
  in HTTP mode — validates JWTs, loads the full user record (auth +
  profile) in one store read, sets `current_user` ContextVar. The
  SDK reads `current_user.get()` for user identity and profile data.

- **User management out of the box.** REST admin endpoints
  (`POST /admin/users`, `GET /admin/users`,
  `DELETE /admin/users/{email}`, `POST /admin/tokens`) — user
  registration with optional profile data, listing, revocation,
  and token issuance with no code.

- **Per-user data storage.** `filesystem` store (default) provides
  per-user JSON under XDG-compliant paths. Custom stores plug in
  via module path. The SDK accesses data through `get_store()`.

- **Typed profile with Pydantic.** Apps register a profile model
  via `register_profile()`. Profile data is validated at registration
  and hydrated as a typed object on `current_user.get().profile`.

- **App CLI factories.** `create_mcp_cli()` gives you `serve` and
  `stdio` commands. `create_admin_cli()` gives you `connect`,
  `users`, `tokens`, and `health`. The admin CLI generates typed
  flags from the profile model automatically.

- **Deployable anywhere.** Standard container image, any platform.
  Or use [gapp](https://github.com/echomodel/gapp) for rapid
  deployment to serverless Cloud Run.

**Install:**
```bash
pip install git+https://github.com/echomodel/mcp-app.git
```

### Alternative: FastMCP (manual wiring)

If the user prefers direct control, already has a FastMCP-based
app, or has requirements that mcp-app doesn't cover, use FastMCP
directly with the `mcp` package. The architecture rules (SDK-first,
thin tools, XDG paths) apply equally to both approaches.

**When to suggest FastMCP directly:**
- User explicitly asks for it
- Existing codebase already uses FastMCP and migration isn't wanted
- Need for custom transport or protocol extensions
- stdio-only tool with no deployment plans
- User wants full control over the ASGI middleware stack

## Modes

This skill operates in three modes depending on what the user
needs. Determine the mode from the user's request and the state
of the current working directory.

### Mode 1: Greenfield — Build a New Solution

User wants to create a new MCP server or web API from scratch.

**First, help them decide: local or remote?**

**Local MCP server (stdio)** makes sense when:
- Works with local files, code, git repos, or system config
- Manages the local workstation or development environment
- Needs fast, low-latency interaction
- Single user, single machine

**Remote MCP server (HTTP)** makes sense when:
- Accesses cloud data or external APIs
- Needs to be available from multiple devices
- Needs multi-user support
- Benefits from always-on availability

Based on this, the solution targets stdio only (simpler — no auth,
no deployment) or stdio + HTTP (needs auth, deployment planning).
Both follow the same repo structure.

### Mode 2: Migration — Port an Existing App

1. Read the existing codebase to understand current structure
2. Walk through the Compliance Checklist, noting conformance
3. Propose a migration plan in priority order
4. Execute with the user's approval
5. Run the checklist again to verify

### Mode 3: Review — Evaluate Against Standards

Run the Compliance Checklist and present results as a table.
For each failure, explain what's wrong and the fix.

## Compliance Checklist

### Structure
- [ ] Three-layer architecture: `sdk/`, `mcp/`, optional `cli/`
- [ ] All business logic in `sdk/`
- [ ] MCP tools are thin (one-liners calling SDK methods)
- [ ] `APP_NAME` constant in `__init__.py`
- [ ] `pyproject.toml` with correct dependencies and entry points

### MCP Server (mcp-app)
- [ ] `mcp-app.yaml` present with `name` and `tools` fields
- [ ] Tools module contains plain async functions (no decorators)
- [ ] Identity middleware runs by default (no config needed)
- [ ] Tool docstrings are clear and user-centric

### MCP Server (FastMCP — if not using mcp-app)
- [ ] Uses `mcp` package (FastMCP) with `stateless_http=True`
- [ ] DNS rebinding protection disabled for Cloud Run
- [ ] `mcp.run()` for stdio, `app` variable for HTTP (uvicorn)

### User Identity and Profile
- [ ] SDK reads `current_user.get()` for identity — `.email` and `.profile`
- [ ] Profile model registered via `register_profile()` if app needs per-user data
- [ ] Profile validated with Pydantic at registration time
- [ ] Per-user data scoping works in both stdio and HTTP modes

### App CLI and Entry Points
- [ ] `create_mcp_cli(APP_NAME)` for serve/stdio commands
- [ ] `create_admin_cli(APP_NAME)` for user management
- [ ] Entry points in `pyproject.toml` for all CLIs
- [ ] Profile model drives typed admin CLI flags

### Paths and Environment Variables
- [ ] XDG path resolver functions in SDK (data, config, cache)
- [ ] Each resolver checks env var override first, then XDG fallback
- [ ] No hardcoded absolute paths in code
- [ ] `SIGNING_KEY` required for HTTP (no default)

### Testing
- [ ] SDK unit tests in `tests/unit/`
- [ ] Full-stack HTTP tests using httpx ASGI transport
- [ ] No mocks unless needed for network I/O
- [ ] Tests use temp dirs and env vars for isolation

### Documentation
- [ ] README.md with quick start, deployment, config
- [ ] CONTRIBUTING.md with architecture and testing standards
- [ ] CLAUDE.md: `@README.md` and `@CONTRIBUTING.md`
- [ ] `.gemini/settings.json` with context file declarations

### Deployment Readiness
- [ ] Deployable as a standard container image
- [ ] Optionally, `gapp.yaml` for rapid Cloud Run deployment
- [ ] `SIGNING_KEY` set as environment variable (no default)

## Repository Structure

### Single-package (recommended for new apps)

```
my-solution/
  my_solution/
    __init__.py       # APP_NAME, mcp_cli, admin_cli, optional Profile
    sdk/
      core.py         # Business logic — ALL behavior here
    mcp/
      tools.py        # Pure async functions calling SDK
    cli/              # Optional — app-specific Click commands
      main.py
  tests/
    unit/
  mcp-app.yaml
  pyproject.toml
```

### Multi-package (when SDK, MCP, CLI have different dependencies)

Some apps separate SDK, MCP server, and CLI into independent
installable packages so users only install what they need:

```
my-solution/
  sdk/
    my_solution/      # my-solution-sdk package
      __init__.py     # APP_NAME
      core.py
    pyproject.toml
  mcp/
    my_solution_mcp/  # my-solution-mcp package
      __init__.py     # mcp_cli, admin_cli, optional Profile
      tools.py
    pyproject.toml    # depends on my-solution-sdk, mcp-app
  cli/
    my_solution_cli/  # my-solution package (CLI)
      main.py
    pyproject.toml    # depends on my-solution-sdk
  mcp-app.yaml
  tests/
```

In multi-package repos, the mcp-app integration (`mcp_cli`,
`admin_cli`, `register_profile`) goes in the MCP package's
`__init__.py` — that's where mcp-app is a dependency.

### __init__.py — the app's identity

**API-proxy app** (needs per-user credentials):

```python
# my_solution/__init__.py
APP_NAME = "my-solution"

from pydantic import BaseModel
from mcp_app.context import register_profile
from mcp_app.cli import create_admin_cli, create_mcp_cli

class Profile(BaseModel):
    token: str  # whatever the app needs per-user

register_profile(Profile, expand=True)

mcp_cli = create_mcp_cli(APP_NAME)
admin_cli = create_admin_cli(APP_NAME)
```

**Data-owning app** (no per-user credentials — just identity):

```python
# my_solution/__init__.py
APP_NAME = "my-solution"

from mcp_app.cli import create_admin_cli, create_mcp_cli

# No Profile model — data-owning apps don't need per-user
# profile data. User identity comes from current_user.get().email.
# App data lives in the store via get_store() or however the app
# chooses to manage its own data storage.

mcp_cli = create_mcp_cli(APP_NAME)
admin_cli = create_admin_cli(APP_NAME)
```

### pyproject.toml entry points

```toml
[project]
name = "my-solution"
dependencies = ["mcp-app"]

[project.scripts]
my-solution = "my_solution.cli:cli"       # app's own CLI (optional)
my-solution-mcp = "my_solution:mcp_cli"   # serve, stdio
my-solution-admin = "my_solution:admin_cli" # connect, users, health
```

One `pipx install my-solution` gives three commands.

### Rules

- **SDK first.** All behavior lives in the SDK. MCP and CLI are
  thin wrappers.
- **No business logic in MCP tools or CLI commands.**
- **If you're writing logic in a tool or command, stop and move it
  to SDK.**

## MCP Server Setup

### With mcp-app (recommended)

**mcp-app.yaml — minimal:**

```yaml
name: my-solution
tools: my_solution.mcp.tools
```

Only `name` and `tools` are required. Store defaults to `filesystem`.
Identity middleware runs automatically in HTTP mode.

For stdio, add:

```yaml
stdio:
  user: "local"
```

**Tools module — plain async functions:**

```python
# my_solution/mcp/tools.py
from my_solution.sdk.core import MySDK

sdk = MySDK()

async def do_thing(param: str) -> dict:
    """Do the thing for the current user."""
    return sdk.do_thing(param)
```

**Run (development, from repo directory):**
```bash
mcp-app serve   # HTTP mode
mcp-app stdio   # stdio mode (reads yaml from cwd)
```

**Run (installed app, from anywhere):**
```bash
my-solution-mcp serve
my-solution-mcp stdio --user local
```

### With FastMCP (alternative)

```python
# my_solution/mcp/server.py
from mcp.server.fastmcp import FastMCP
from my_solution import APP_NAME
from my_solution.sdk.core import MySDK

mcp = FastMCP(APP_NAME, stateless_http=True, json_response=True)
mcp.settings.transport_security.enable_dns_rebinding_protection = False

sdk = MySDK()

@mcp.tool()
async def do_thing(param: str) -> dict:
    """Do the thing."""
    return sdk.do_thing(param)

app = mcp.streamable_http_app()  # For uvicorn HTTP mode

def run_server():
    mcp.run()             # For stdio mode
```

## User Identity and Profile

### How it works

In HTTP mode, identity middleware validates the JWT, loads the full
user record from the store (auth + profile in one read), and sets
`current_user` ContextVar. In stdio mode, the CLI loads the user
record from the store using `stdio.user` from yaml.

The SDK reads it:

```python
from mcp_app.context import current_user

user = current_user.get()
user.email       # "alice@example.com" (HTTP) or "local" (stdio)
user.profile     # typed Pydantic model or raw dict
```

### Two app patterns — same framework, different SDK reads

**Data-owning** (owns user data — food logs, notes):

```python
from mcp_app.context import current_user
from mcp_app import get_store

class MySDK:
    def save_entry(self, data):
        user = current_user.get()
        store = get_store()
        store.save(user.email, "entries/today", data)
```

The SDK reads `current_user.get().email` for user identity. How
it stores data is the app's choice — `get_store()` provides
mcp-app's per-user key-value store, but the SDK can also manage
its own storage (XDG paths, custom databases, etc.) using the
email as the scoping key.

**API-proxy** (wraps external API — financial data, task management):

```python
from mcp_app.context import current_user
import httpx

class MySDK:
    def list_items(self):
        user = current_user.get()
        token = user.profile.token  # typed via Pydantic
        resp = httpx.get("https://api.example.com/items",
                         headers={"Authorization": f"Bearer {token}"})
        return resp.json()
```

Both use `current_user.get()`. The middleware is the same (identity
only). The SDK decides what to read from the user context.

### Profile registration

The app declares its per-user profile shape:

```python
# my_solution/__init__.py
from pydantic import BaseModel
from mcp_app.context import register_profile

class Profile(BaseModel):
    token: str

register_profile(Profile, expand=True)
```

`expand=True` generates typed CLI flags (`--token`). `expand=False`
accepts the profile as a JSON blob or `@file`.

### User management

**Three CLI entry points per app:**

```bash
# App CLI — the app itself (if it has one)
my-solution do-something

# MCP server
my-solution-mcp serve
my-solution-mcp stdio --user local

# Admin — user management
my-solution-admin connect local
my-solution-admin users add alice --token xxx
my-solution-admin users list
my-solution-admin users revoke alice
my-solution-admin health
```

`connect local` writes to the local filesystem store. `connect <url>`
manages users on a deployed instance via HTTP admin API.

**Generic CLI (no app installed):**

```bash
mcp-app setup https://my-app.run.app --signing-key xxx
mcp-app users add alice --profile '{"token": "xxx"}'
```

Remote only. No typed flags (profile model isn't loaded).

### Environment variables

| Var | Required | Default | Purpose |
|-----|----------|---------|---------|
| `SIGNING_KEY` | For HTTP | none — required | JWT signing |
| `JWT_AUD` | No | None | Token audience |
| `APP_USERS_PATH` | No | XDG default | Data directory |

## Testing and Validation

After building the solution, write tests and validate both transports.
This is not optional — do it before the compliance dashboard.

### Step 1: SDK unit tests

Test business logic directly with env var isolation:

```python
import os
import pytest

@pytest.fixture(autouse=True)
def isolated_data(tmp_path):
    os.environ["MY_SOLUTION_DATA"] = str(tmp_path / "data")
    os.environ["MY_SOLUTION_CONFIG"] = str(tmp_path / "config")
    yield
    del os.environ["MY_SOLUTION_DATA"]
    del os.environ["MY_SOLUTION_CONFIG"]

def test_saves_entry(tmp_path):
    sdk = MySDK()
    result = sdk.save_entry({"item": "apple"})
    assert result["success"]
```

### Step 2: Full-stack HTTP validation

Validate the entire ASGI stack in-memory using httpx's ASGI
transport. No server process, no port, no Docker. httpx is a
dependency of mcp-app — the solution gets it for free.

**If it works here, it works in uvicorn, it works in Docker.**

```python
import httpx
import jwt as pyjwt
from datetime import datetime, timezone, timedelta
from mcp_app.bootstrap import build_app

@pytest.fixture
def app_client(tmp_path):
    os.environ["APP_USERS_PATH"] = str(tmp_path / "users")
    os.environ["SIGNING_KEY"] = "test-key"
    app, mcp, store, config = build_app()
    transport = httpx.ASGITransport(app=app)
    return httpx.AsyncClient(transport=transport, base_url="http://test")

@pytest.mark.asyncio
async def test_register_and_call_tool(app_client):
    admin_token = pyjwt.encode(
        {"sub": "admin", "scope": "admin",
         "iat": datetime.now(timezone.utc),
         "exp": datetime.now(timezone.utc) + timedelta(minutes=5)},
        "test-key", algorithm="HS256",
    )
    headers = {"Authorization": f"Bearer {admin_token}"}

    resp = await app_client.post(
        "/admin/users",
        json={"email": "user@example.com",
              "profile": {"token": "test-api-key"}},
        headers=headers,
    )
    assert resp.status_code == 200
    assert "token" in resp.json()
```

### Step 3: stdio validation

**Development (from repo directory):**
```bash
claude mcp add my-solution -- mcp-app stdio
```

**Installed app:**
```bash
claude mcp add my-solution -- my-solution-mcp stdio --user local
```

### Step 4: Run tests

```bash
pytest tests/unit/ -v
```

All tests must pass before the compliance dashboard.

### When to stub

- **Network I/O** — stub HTTP clients, external API calls
- **Cloud CLIs** — mock at the SDK function boundary
- **Everything else** — real code, real files, real config

## Deployment

The solution is a standard Python app. It deploys as a container
anywhere.

### Option 1: gapp (fastest path to Cloud Run)

[gapp](https://github.com/echomodel/gapp) auto-detects
`mcp-app.yaml` and handles Dockerfile generation, secrets, and
data volumes:

```yaml
# gapp.yaml
public: true
env:
  - name: SIGNING_KEY
    secret:
      generate: true
  - name: APP_USERS_PATH
    value: "{{SOLUTION_DATA_PATH}}/users"
```

```bash
gapp deploy
```

### Option 2: Docker (any container platform)

```dockerfile
FROM python:3.11-slim
WORKDIR /app
COPY . /app
RUN pip install -e .
EXPOSE 8080
CMD ["mcp-app", "serve"]
```

```bash
docker build -t my-solution .
docker run -p 8080:8080 -e SIGNING_KEY=your-key my-solution

# Deploy to Google Cloud Run
gcloud run deploy my-solution \
  --source . \
  --allow-unauthenticated \
  --set-env-vars SIGNING_KEY=your-key
```

The Dockerfile works on any container platform. Cloud Run is shown
as a concrete example.

### After deployment: MCP client registration

**Claude Code (stdio — installed app):**
```bash
claude mcp add my-solution -- my-solution-mcp stdio --user local
```

**Claude Code (HTTP — remote):**
```bash
claude mcp add --transport http my-solution \
  https://your-service.run.app/ \
  --header "Authorization: Bearer USER_TOKEN"
```

**Claude.ai / Claude mobile / Claude Code (remote via URL):**
```
https://your-service.run.app/?token=USER_TOKEN
```

Remote MCP servers added through Claude.ai are available across all
Claude clients — web, mobile app, and Claude Code.

### After deployment: user management

```bash
# Remote — via admin CLI
my-solution-admin connect https://your-service.run.app --signing-key xxx
my-solution-admin users add alice --token api-key-xxx

# Or via generic CLI
mcp-app setup https://your-service.run.app --signing-key xxx
mcp-app users add alice --profile '{"token": "api-key-xxx"}'
```

The token returned is what the user puts in their MCP client config.

## Gitignore

```gitignore
# Python
__pycache__/
*.py[cod]
*.egg-info/
build/
dist/
.venv/
venv/
.eggs/

# Environment
.env

# Testing
.pytest_cache/

# Claude Code
.claude/

# Gemini (except settings.json)
.gemini/*
!.gemini/settings.json

# OS
.DS_Store

# Logs
*.log
```

## Documentation

### README.md
Covers: why the repo exists, quick start, deployment, CLI, config.

### CONTRIBUTING.md
Covers: architecture, testing, conventions, how to add features.

### Agent context files

**CLAUDE.md:**
```markdown
@README.md
@CONTRIBUTING.md
```

**`.gemini/settings.json`:**
```json
{
  "context": {
    "fileName": ["README.md", "CONTRIBUTING.md"]
  }
}
```

## Final Step: Compliance Dashboard

**Always conclude with this.** Run the Compliance Checklist and
present results:

```
## Solution Compliance Dashboard: {APP_NAME}

| Category | Item | Status |
|----------|------|--------|
| Structure | SDK layer contains all business logic | ✅ |
| Structure | MCP tools are thin wrappers | ✅ |
| MCP | mcp-app.yaml present | ✅ |
| Identity | current_user accessible, profile typed | ✅ |
| CLI | Entry points for mcp + admin | ❌ |
| Testing | SDK unit tests pass | ✅ |
| Testing | Full-stack HTTP tests pass | ⚠️ |
| Testing | stdio validated | ❌ |
| Deploy | Dockerfile or gapp.yaml present | ❌ |

✅ = conforms  ❌ = missing/wrong  ⚠️ = partial
```

After presenting the dashboard:

1. If there are ❌ or ⚠️ items: "Want me to fix these?"
2. If all ✅: "This solution is ready for deployment."
