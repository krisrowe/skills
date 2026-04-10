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
`mcp-app serve`. No framework imports in your tool code.

**What the user gets with mcp-app:**

- **Zero framework code in tools.** Tools are plain async functions.
  No decorators, no FastMCP imports, no framework coupling in
  business code. mcp-app discovers public async functions from the
  module declared in `tools:` and registers them automatically —
  function names become tool names, docstrings become descriptions,
  type hints become schemas.

- **User management out of the box.** In HTTP mode, mcp-app
  automatically mounts REST admin endpoints (`POST /admin/users`,
  `GET /admin/users`, `DELETE /admin/users/{email}`,
  `POST /admin/tokens`) — user registration, listing, revocation,
  and token issuance with no code. Adding `user-identity` to the
  `middleware:` list protects the MCP tools with JWT auth so only
  registered users can call them. With raw FastMCP, you wire all
  of this yourself (manual middleware stacking, DataStoreAuthAdapter,
  admin endpoint wiring).

- **Per-user data storage.** The `store:` config gives you a data
  store abstraction. `filesystem` (the default) stores per-user
  JSON under XDG-compliant paths. Custom stores (database, cloud
  storage) plug in via a module path. The SDK accesses data through
  `get_store()` — no direct file I/O to manage.

- **One command to run.** `mcp-app serve` starts the HTTP server.
  No uvicorn invocation, no ASGI app variable, no DNS rebinding
  config. For stdio mode, `mcp-app stdio` reads the same yaml
  and runs over stdin/stdout.

- **Deployable anywhere.** Standard container image, any platform.
  Or use [gapp](https://github.com/echomodel/gapp) for rapid
  deployment to serverless Cloud Run — gapp auto-detects
  `mcp-app.yaml` and handles Dockerfile generation, secrets, and
  data volumes.

**The tradeoff:** mcp-app is opinionated. It handles auth, admin,
stores, and tool discovery its way. If the user needs custom
transports, non-standard protocol extensions, or wants full control
over the ASGI stack, FastMCP gives them that.

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

Follow the full guide below from Repository Structure onward.

### Mode 2: Migration — Port an Existing App

User has an existing app and wants to port it to follow these
conventions. Steps:

1. Read the existing codebase to understand current structure
2. Walk through the Compliance Checklist below, noting what
   already conforms and what needs to change
3. Propose a migration plan — what to move, what to delete,
   what to add — in priority order
4. Execute the migration with the user's approval
5. Run the checklist again to verify compliance

### Mode 3: Review — Evaluate Against Standards

User wants a compliance check of their existing solution.

Run the **Compliance Checklist** below and present results as
a table:

| Item | Status | Notes |
|------|--------|-------|
| SDK layer contains all business logic | ✅ | |
| MCP tools are thin wrappers | ✅ | |
| APP_NAME constant in __init__.py | ❌ | Missing |
| ... | ... | ... |

For each ❌, explain what's wrong and what the fix would be.
Ask the user if they want to fix the issues.

## Compliance Checklist

Use this for Mode 2 (migration) and Mode 3 (review):

### Structure
- [ ] Three-layer architecture: `sdk/`, `mcp/`, optional `cli/`
- [ ] All business logic in `sdk/` — no logic in MCP tools or CLI commands
- [ ] MCP tools are thin (one-liners calling SDK methods)
- [ ] `APP_NAME` constant in `__init__.py`, used everywhere
- [ ] `pyproject.toml` with correct dependencies

### MCP Server (mcp-app)
- [ ] `mcp-app.yaml` present with `name` and `tools` fields (`store` defaults to `filesystem`)
- [ ] Tools module contains plain async functions (no decorators)
- [ ] `mcp-app serve` starts the HTTP server
- [ ] Tool docstrings are clear and user-centric

### MCP Server (FastMCP — if not using mcp-app)
- [ ] Uses `mcp` package (FastMCP) with `stateless_http=True`
- [ ] DNS rebinding protection disabled for Cloud Run
- [ ] `mcp.run()` for stdio, `app` variable for HTTP (uvicorn)
- [ ] All tools have clear docstrings

### Multi-User Auth (only if solution needs it)
- [ ] Middleware configured in `mcp-app.yaml`: `user-identity` for data-owning, `bearer-proxy` or `google-oauth2-proxy` for API-proxy (or manual wiring for FastMCP)
- [ ] SDK reads `current_user_id` from context — never from request directly
- [ ] Per-user data scoping works in both stdio and HTTP modes

### Paths and Environment Variables
- [ ] XDG path resolver functions in SDK (data, config, cache)
- [ ] Each resolver checks env var override first, then XDG fallback with APP_NAME
- [ ] No hardcoded absolute paths in code
- [ ] Tests use the same env var overrides for isolation

### Testing
- [ ] Sociable unit tests in `tests/unit/`
- [ ] No mocks unless explicitly justified
- [ ] Tests use temp dirs and env vars for isolation
- [ ] Test names describe scenario + outcome

### Documentation
- [ ] README.md: why the repo exists, quick start, deployment, CLI overview, config, dev guide
- [ ] CONTRIBUTING.md: architecture, testing standards, conventions, how to add features
- [ ] CLAUDE.md: thin, `@import README.md` and `@import CONTRIBUTING.md`
- [ ] `.gemini/settings.json`: `context.fileName` pointing to README.md and CONTRIBUTING.md
- [ ] `.gitignore` follows baseline (see Gitignore section)
- [ ] No stale references to removed features

### Deployment Readiness
- [ ] `mcp-app.yaml` present (mcp-app) or `app` variable in `server.py` (FastMCP)
- [ ] Deployable as a standard container image (Dockerfile or buildpack)
- [ ] Optionally, `gapp.yaml` for rapid Cloud Run deployment via gapp
- [ ] All secrets declared in deployment config (env vars, not hardcoded)

## Repository Structure

Every solution follows a three-layer architecture:

```
my-solution/
  my_solution/
    __init__.py       # APP_NAME constant
    sdk/              # Business logic — ALL behavior lives here
      core.py         # Main SDK class
      paths.py        # XDG path resolvers
    mcp/
      tools.py        # Tool definitions — thin, calls SDK
    cli/              # Optional — Click commands, calls SDK
      main.py
  tests/
    unit/             # Sociable unit tests, no mocks
  mcp-app.yaml        # If using mcp-app
  pyproject.toml
  gapp.yaml           # Optional — only if deploying with gapp
```

### Rules

- **SDK first.** All behavior lives in the SDK. MCP and CLI are
  thin wrappers that call SDK methods and format output.
- **No business logic in MCP tools.** Tools call SDK methods.
- **No business logic in CLI commands.** Commands call SDK methods.
- **If you're writing logic in a tool or command, stop and move it
  to SDK.**

### APP_NAME constant

Define once, use everywhere:

```python
# my_solution/__init__.py
APP_NAME = "my-solution"
```

Used for server name, data store paths, and XDG directory naming.

## MCP Server Setup

### With mcp-app (recommended)

**mcp-app.yaml:**

```yaml
name: my-solution
store: filesystem
middleware:
  - user-identity
tools: my_solution.mcp.tools
```

| Field | Required | Default | Purpose |
|-------|----------|---------|---------|
| `name` | Yes | — | MCP server name, data store paths, XDG directory naming |
| `tools` | Yes | — | Python module path to discover tools from. All public async functions in this module become MCP tools automatically. |
| `store` | No | `filesystem` | Data store backend. `filesystem` gives per-user JSON storage under XDG paths. Use a dotted module path (e.g., `my_app.stores.MyStore`) for a custom backend (database, cloud storage, etc.). |
| `middleware` | No | none | Array of middleware. Built-in: `user-identity` (data-owning), `bearer-proxy` (API-proxy with static tokens), `google-oauth2-proxy` (API-proxy with Google APIs). Custom middleware via dotted module path. Omit entirely for no auth. |

**Tools module — plain async functions:**

```python
# my_solution/mcp/tools.py
from my_solution.sdk.core import MySDK

sdk = MySDK()

async def do_thing(param: str) -> dict:
    """Do the thing for the current user."""
    return sdk.do_thing(param)
```

No decorators, no framework imports. mcp-app discovers all public
async functions and registers them as MCP tools. Function names
become tool names. Docstrings become descriptions. Type hints
become schemas.

**Run:**
```bash
mcp-app serve          # HTTP mode
python -m my_solution  # stdio mode (if you add __main__.py)
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

### stdio vs HTTP

- **stdio** (local): single user, no auth, fast
- **HTTP** (deployed): multi-user, auth via middleware, always-on

Both use the same tools and SDK. The only difference is how the
server starts and whether auth is present.

## XDG Path Convention

Solutions use XDG Base Directory Specification for local paths:

```python
# my_solution/sdk/paths.py
import os
from pathlib import Path
from my_solution import APP_NAME

def get_data_dir() -> Path:
    """Data directory (logs, user data, catalogs)."""
    env = os.environ.get("MY_SOLUTION_DATA")
    if env:
        return Path(env).expanduser().resolve()
    base = Path(os.environ.get("XDG_DATA_HOME", Path.home() / ".local" / "share"))
    return base / APP_NAME

def get_config_dir() -> Path:
    """Config directory (settings, app.yaml)."""
    env = os.environ.get("MY_SOLUTION_CONFIG")
    if env:
        return Path(env).expanduser().resolve()
    base = Path(os.environ.get("XDG_CONFIG_HOME", Path.home() / ".config"))
    return base / APP_NAME
```

The env var overrides serve two purposes:
1. **Deployment** — map to cloud storage mount paths
2. **Testing** — point to temp dirs for isolation

## CLI Layer

Use Click for CLI commands. The CLI is a thin wrapper that calls
SDK methods — same rule as MCP tools.

### CLI scope — discuss with the user

**Recommended starting point:** minimal CLI for management and
security-sensitive operations. Add CLI equivalents of MCP tools
only when the user has a concrete need (scripting, automation).

### SDK returns JSON, always

SDK methods return dicts (JSON-serializable). Both MCP tools and
CLI commands call the same SDK methods:

```python
# SDK
def log_food(self, entries) -> dict:
    return {"success": True, "date": "2026-03-25", "entries_added": 2}

# MCP tool — returns directly
async def log_meal(food_entries: list) -> dict:
    """Log a meal."""
    return sdk.log_food(food_entries)

# CLI — formats for humans
@cli.command()
@click.option("--json", "as_json", is_flag=True)
def log(food, as_json):
    result = sdk.log_food(food)
    if as_json:
        click.echo(json.dumps(result, indent=2))
    else:
        click.echo(f"Logged {result['entries_added']} entries")
```

## Multi-User Auth

### With mcp-app (recommended)

Configure middleware in `mcp-app.yaml`:

```yaml
name: my-solution
store: filesystem
middleware:
  - user-identity       # For data-owning apps
tools: my_solution.mcp.tools
```

Three built-in middleware aliases cover the common patterns:

| Alias | Use case |
|-------|----------|
| `user-identity` | Data-owning apps. Validates JWT, sets `current_user_id` ContextVar. |
| `bearer-proxy` | API-proxy apps with static tokens (PATs, API keys). Validates JWT, swaps for stored backend token, rewrites Authorization header. |
| `google-oauth2-proxy` | API-proxy apps wrapping Google APIs. Same as bearer-proxy but refreshes expired OAuth2 access tokens automatically. |

`bearer-proxy` and `google-oauth2-proxy` are tracked in
echomodel/mcp-app#8 — not yet implemented. For now, API-proxy
apps use a custom middleware class referenced by module path, or
FastMCP with manual middleware wiring.

The SDK reads user identity from context:

```python
from mcp_app.context import current_user_id
user = current_user_id.get()  # "default" (stdio) or "user@example.com" (HTTP)
```

### With FastMCP (alternative — manual auth wiring)

Wire auth manually using mcp-app's auth components:

```python
from mcp_app import FileSystemUserDataStore, DataStoreAuthAdapter
from mcp_app.admin import create_admin_app
from mcp_app.middleware import JWTMiddleware
from mcp_app.verifier import JWTVerifier
from starlette.applications import Starlette
from starlette.routing import Mount

store = FileSystemUserDataStore(app_name=APP_NAME)
auth_store = DataStoreAuthAdapter(store)
verifier = JWTVerifier(auth_store)
admin_app = create_admin_app(auth_store)
inner = mcp.streamable_http_app()
wrapped = JWTMiddleware(inner, verifier)

app = Starlette(routes=[
    Mount("/admin", app=admin_app),
    Mount("/", app=wrapped),
])
```

This is what `mcp-app serve` does internally via `build_app()`.
Prefer mcp-app.yaml unless you need custom ASGI composition.

### Environment variables

| Var | Required | Default | Purpose |
|-----|----------|---------|---------|
| `SIGNING_KEY` | For HTTP | `"dev-key"` | JWT signing |
| `JWT_AUD` | No | None | Token audience |
| `APP_USERS_PATH` | No | XDG default | Data directory |

## Testing and Validation

After building the solution, write tests and validate both transports.
This is not optional — do it before the compliance dashboard.

### Step 1: SDK unit tests

Test business logic directly. Isolate via env vars pointing to temp
dirs — same env vars the solution's XDG resolvers read, same ones
deployment maps to cloud storage.

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

def test_logs_food_to_date_directory(tmp_path):
    sdk = MySDK()
    result = sdk.log_food([{"food_name": "apple"}])
    assert result["success"]
```

No mocks unless needed for network I/O or cloud CLIs. Real file I/O
to temp dirs. Real JSON parsing. Real config resolution.

### Step 2: Full-stack HTTP validation (mcp-app)

Validate the entire ASGI stack in-memory using httpx's ASGI transport.
No server process, no port, no Docker. httpx is already a dependency
of mcp-app — the solution gets it for free.

`build_app()` returns the same ASGI app that `mcp-app serve` gives to
uvicorn. Give it to httpx instead and the full stack runs in-process:
tool discovery, middleware, admin endpoints, store wiring.

**If it works here, it works in uvicorn, it works in Docker.**

```python
import os
import pytest
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

def _admin_token():
    return pyjwt.encode(
        {"sub": "admin", "scope": "admin",
         "iat": datetime.now(timezone.utc),
         "exp": datetime.now(timezone.utc) + timedelta(minutes=5)},
        "test-key", algorithm="HS256",
    )

@pytest.mark.asyncio
async def test_register_user_and_list(app_client):
    headers = {"Authorization": f"Bearer {_admin_token()}"}
    resp = await app_client.post(
        "/admin/users",
        json={"email": "user@example.com"},
        headers=headers,
    )
    assert resp.status_code == 200
    assert "token" in resp.json()

    resp = await app_client.get("/admin/users", headers=headers)
    assert any(u["email"] == "user@example.com" for u in resp.json())
```

Write tests that cover:
- Tool discovery (tools from the module are reachable)
- Auth flow (register user, get token, call tool with token)
- SDK logic through the tool layer (end-to-end)
- Store reads/writes via tools

### Step 3: stdio validation

Register with an MCP client — it manages the process lifecycle:

```bash
claude mcp add my-solution -- mcp-app stdio
```

No background server, no port management, no cleanup. Call tools
through the agent and verify they work.

Requires `stdio.user` in `mcp-app.yaml`:

```yaml
stdio:
  user: "local"
```

`mcp-app stdio` refuses to start without this — there are no
silent defaults for user identity.

### Step 4: Run tests

```bash
pytest tests/unit/ -v
```

All tests must pass before proceeding to the compliance dashboard
or deployment.

### When to stub

- **Network I/O** — stub HTTP clients, external API calls
- **Cloud CLIs** — mock at the SDK function boundary
- **Local CLI tools** (git, etc.) — let them run for real in temp dirs
- **Everything else** — real code, real files, real config

## Deployment

The solution is a standard Python app. It deploys as a container
anywhere. mcp-app doesn't care where it runs — it serves an ASGI
app on a port.

### Option 1: gapp (fastest path to Cloud Run)

[gapp](https://github.com/echomodel/gapp) auto-detects `mcp-app.yaml`
and handles Dockerfile generation, secrets, and data volumes:

```yaml
# gapp.yaml — minimal config
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

No Dockerfile to write. gapp generates one with
`CMD ["mcp-app", "serve"]`.

### Option 2: Docker (any container platform)

Write a Dockerfile and deploy to Cloud Run, Fly, Railway, or any
platform that runs containers:

```dockerfile
FROM python:3.11-slim
WORKDIR /app
COPY . /app
RUN pip install -e .
EXPOSE 8080
CMD ["mcp-app", "serve"]
```

```bash
# Build and test locally
docker build -t my-solution .
docker run -p 8080:8080 -e SIGNING_KEY=dev-key my-solution
# Visit http://localhost:8080/admin/users to verify

# Deploy to Google Cloud Run (source deploy — builds in the cloud)
gcloud run deploy my-solution \
  --source . \
  --allow-unauthenticated \
  --set-env-vars SIGNING_KEY=your-key

# Deploy to Google Cloud Run (from pre-built image)
gcloud builds submit --tag gcr.io/your-project/my-solution
gcloud run deploy my-solution \
  --image gcr.io/your-project/my-solution \
  --allow-unauthenticated \
  --set-env-vars SIGNING_KEY=your-key
```

The Dockerfile is standard — it works on any platform that runs
containers. Cloud Run is shown here as a concrete example. The
solution is not locked to any cloud provider.

Set `SIGNING_KEY` and `APP_USERS_PATH` as environment variables on
your platform. `mcp-app serve` starts on port 8080 by default.

### After deployment: MCP client registration

**Claude Code (stdio — local):**
```bash
claude mcp add my-solution -- mcp-app stdio
```

**Claude Code (HTTP — remote):**
```bash
claude mcp add --transport http my-solution \
  https://your-service.run.app/ \
  --header "Authorization: Bearer USER_TOKEN"
```

Note: Claude Code does not currently support env var interpolation
in HTTP headers. The token is stored in cleartext in the local
config. For stdio servers, use `-e KEY=value` to pass secrets via
environment variables instead.

**Claude.ai / Claude mobile / Claude Code (remote via URL):**
```
https://your-service.run.app/?token=USER_TOKEN
```

Remote MCP servers added through Claude.ai are available across all
Claude clients — web, mobile app, and Claude Code — without separate
configuration for each.

**Gemini CLI (remote):**
```json
{
  "mcpServers": {
    "my-solution": {
      "url": "https://your-service.run.app/",
      "headers": {
        "Authorization": "Bearer USER_TOKEN"
      }
    }
  }
}
```

### After deployment: user management

Register users via the admin API:
```bash
mcp-app set-base-url https://your-service.run.app --signing-key YOUR_KEY
mcp-app users add user@example.com
```

The token returned is what the user puts in their MCP client config.

## Gitignore

Baseline for Python MCP repos:

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

Covers: why the repo exists, quick start, deployment, CLI overview,
configuration, development guide.

### CONTRIBUTING.md

Covers: architecture, testing standards, conventions, how to add
features, security considerations.

### Agent context files

**CLAUDE.md:**
```markdown
@import README.md
@import CONTRIBUTING.md
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

**Always conclude with this** — whether greenfield, migration, or
review. Run the Compliance Checklist and present results:

```
## Solution Compliance Dashboard: {APP_NAME}

| Category | Item | Status |
|----------|------|--------|
| Structure | SDK layer contains all business logic | ✅ |
| Structure | MCP tools are thin wrappers | ✅ |
| MCP | mcp-app.yaml present (or FastMCP configured) | ✅ |
| Auth | Middleware configured | ❌ |
| Testing | SDK unit tests pass | ✅ |
| Testing | Full-stack HTTP tests pass | ⚠️ |
| Testing | stdio validated | ❌ |
| Deploy | Dockerfile or gapp.yaml present | ❌ |
| ... | ... | ... |

✅ = conforms  ❌ = missing/wrong  ⚠️ = partial
```

After presenting the dashboard:

1. If there are ❌ or ⚠️ items: "Want me to fix these?"
2. If all ✅: "This solution is ready for deployment."
