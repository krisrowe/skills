---
name: mcp-app-admin
description: Operate and manage deployed MCP apps or solutions that use the mcp-app framework. Use when asked to verify a deployment, connect the admin CLI, retrieve a signing key, register or manage users, issue or revoke tokens, update a user's profile with a fresh third-party token for a backend API that the MCP server provides tools for or proxies, test a deployed service end-to-end, configure a deployed MCP server for use in Claude, Gemini, or other agent platforms, or troubleshoot post-deploy auth. Triggers on verify deployment, test the deployed service, manage users, add a user, list users, revoke a user, update a token, refresh a credential, get the signing key, connect the admin CLI, configure MCP client, issue a new token, and similar post-deploy operational tasks on running mcp-app services.
---

# mcp-app Admin

## Overview

This skill covers operating mcp-app solutions after deployment:
retrieving the signing key, connecting the admin CLI, managing
users, verifying the deployment end-to-end, and registering MCP
clients. It applies regardless of how the solution was deployed —
Cloud Run, Docker, bare metal, or any other environment.

The `author-mcp-app` skill, if available, covers building and
structuring solutions. This skill picks up where deployment ends.

## Step 1: Retrieve the Signing Key

The signing key is required for all admin operations. It's stored
wherever the deployment process put it — you need to trace it back
through the deployment tooling and configuration.

### How to find it

**Start with the deployment configuration.** Look at how the
solution was deployed and how `SIGNING_KEY` was configured:

- **gapp.yaml** with `secret: { name: ..., generate: true }` —
  gapp generated and stored the key in GCP Secret Manager.
  Retrieve it using the secret's short name from gapp.yaml:
  ```bash
  gapp secrets get <secret-name> --raw
  ```
  Or: `gapp_secret_get(name="signing-key", plaintext=True)`

- **Terraform** managing the secret (e.g., `random_password`
  resource) — the value is in Terraform state. Retrieve with:
  ```bash
  terraform output -raw signing_key
  ```
  Or if stored in a cloud secret manager by Terraform, read it
  from there (see cloud secret manager below).

- **Cloud secret manager** (GCP Secret Manager, AWS Secrets
  Manager, etc.) — the key was stored there by the deployment
  tool or manually. Read it directly:
  ```bash
  # GCP
  gcloud secrets versions access latest --secret=SECRET_ID --project=PROJECT_ID
  # AWS
  aws secretsmanager get-secret-value --secret-id SECRET_ID
  ```
  To find the secret name, check the deployment config or inspect
  the running service:
  ```bash
  # GCP Cloud Run — shows which secrets are mounted as env vars
  gcloud run services describe SERVICE --region=REGION --format=json \
    | jq '.spec.template.spec.containers[0].env[] | select(.name=="SIGNING_KEY")'
  ```

- **Docker Compose** with a secrets file or env var — check the
  `docker-compose.yml` for the secret source (file path, env var
  reference, or Docker secret).

- **CI/CD secrets** (GitHub Actions, GitLab CI, etc.) — the key
  is stored in the CI platform's secrets store. These are
  write-only from the UI — you can't retrieve them. If this is
  the only copy, generate a new key, update the CI secret, and
  redeploy.

- **Environment variable set manually** — check the process
  environment or the shell profile / systemd unit / supervisor
  config that launched the server.

### If you can't find it

If the signing key is truly lost (e.g., only copy was in a CI
secret you can't read back), generate a new one:

```bash
python3 -c 'import secrets; print(secrets.token_urlsafe(32))'
```

Store it wherever the deployment expects it, redeploy, and
re-register all users (existing tokens become invalid).

## Step 2: Connect the Admin CLI

The admin CLI needs the service URL and signing key. First,
check if the app's own admin CLI is available:

```bash
which my-solution-admin
```

If not found, install the package that provides the admin
entry point. Check the solution's `pyproject.toml` for
`[project.scripts]` to find which package declares
`my-solution-admin`. In single-package repos this is the root
package. In multi-package repos (separate sdk/, mcp/, cli/
directories each with their own pyproject.toml), the admin CLI
is typically in the mcp package:

```bash
# Single-package repo
pipx install git+https://github.com/owner/my-solution.git

# Multi-package repo — install the mcp package
pipx install -e mcp/

# Or from a local clone of a single-package repo
pipx install -e .
```

Then verify the admin entry point is available:
```bash
my-solution-admin --help
```

**App-specific admin CLI** (preferred — has typed profile flags
derived from the app's profile model):
```bash
my-solution-admin connect https://your-service --signing-key xxx
```

**Generic mcp-app CLI** (fallback when the app's admin CLI
can't be installed — e.g., working from a machine without the
app's repo):
```bash
mcp-app setup https://your-service --signing-key xxx
```

If piping the signing key from a secrets tool:
```bash
gapp secrets get signing-key --raw | \
  my-solution-admin connect https://your-service --signing-key-stdin
```

### Verify the connection

```bash
my-solution-admin health
```

Should show `healthy (200)`. If it fails:
- Check the URL (trailing slash may matter)
- Confirm the signing key matches what the deployed service has
- Confirm the service allows unauthenticated HTTP traffic (mcp-app
  handles auth itself — platform-level auth gates like Cloud Run
  IAM must be disabled or set to allow all)

## Step 3: Verify the Deployment

Run these checks in order after connecting:

**1. Liveness** — confirm the process is up:
```bash
curl https://your-service/health
# {"status": "ok"}
```

**2. Admin auth** — confirm the admin connection works:
```bash
my-solution-admin users list
```

**3. Register a test user** (if no users exist yet):
```bash
my-solution-admin users add alice@example.com
# Returns: {"email": "alice@example.com", "token": "..."}
```

For API-proxy apps with a profile model:
```bash
my-solution-admin users add alice@example.com --token api-key-xxx
```

**4. End-to-end user auth** — confirm a user token works by
calling `tools/list`:
```bash
curl -X POST https://your-service/ \
  -H "Authorization: Bearer USER_TOKEN" \
  -H "Content-Type: application/json" \
  -H "Accept: application/json" \
  -d '{"jsonrpc": "2.0", "method": "tools/list", "id": 1}'
```

If this returns the tool list, user auth works, the store loaded
the user record, and tools are wired. The app is fully operational.

## User Management

### Managing users

```bash
# Register a new user
my-solution-admin users add alice@example.com

# Register with profile data (API-proxy apps)
my-solution-admin users add bob@example.com --token api-key-xxx

# List all users
my-solution-admin users list

# Revoke a user (invalidates all their tokens immediately)
my-solution-admin users revoke alice@example.com

# Issue a new token for an existing user
my-solution-admin tokens create alice@example.com
```

### When to use the generic CLI

If the app's own admin CLI (`my-solution-admin`) isn't installed
locally — e.g., you're managing a deployed instance from a machine
that doesn't have the app package — use the generic `mcp-app` CLI:

```bash
mcp-app setup https://your-service --signing-key xxx
mcp-app users add alice@example.com --profile '{"token": "xxx"}'
```

The generic CLI works but doesn't have typed profile flags — you
pass profile data as a JSON string.

### Token lifecycle

- Tokens are long-lived by default (~10 years) because MCP clients
  cannot refresh tokens automatically.
- Revocation is the primary access control — `users revoke` sets a
  cutoff timestamp, and all tokens issued before that moment are
  rejected.
- After revoking, issue a new token with `tokens create` to
  reactivate the user.
- The token from `users add` or `tokens create` is what the user
  configures in their MCP client.

## MCP Client Registration

### stdio (local)

No signing key needed — stdio has no JWT auth.

**CLI registration:**
```bash
claude mcp add my-solution -- my-solution-mcp stdio --user local
gemini mcp add my-solution -- my-solution-mcp stdio --user local
```

**Manual config** (`~/.claude.json` or `~/.gemini/settings.json`):
```json
{
  "mcpServers": {
    "my-solution": {
      "command": "my-solution-mcp",
      "args": ["stdio", "--user", "local"]
    }
  }
}
```

### HTTP (remote)

**CLI registration:**
```bash
claude mcp add --transport http my-solution \
  https://your-service/ \
  --header "Authorization: Bearer USER_TOKEN"
```

**Manual config** (`~/.claude.json` or `~/.gemini/settings.json`):
```json
{
  "mcpServers": {
    "my-solution": {
      "url": "https://your-service/",
      "headers": {
        "Authorization": "Bearer ${MY_SOLUTION_TOKEN}"
      }
    }
  }
}
```

Both Claude Code and Gemini CLI support `${VAR}` expansion —
reference a host environment variable instead of pasting tokens
into config files.

**Claude.ai / Claude mobile (remote via URL):**
```
https://your-service/?token=USER_TOKEN
```

Remote MCP servers added through Claude.ai are available across
all Claude clients — web, mobile, and Claude Code.

## Important Notes

- User management is an mcp-app concern, not a deployment tool
  concern. The admin endpoints and CLI are part of mcp-app.
- Admin tokens are generated locally using the signing key — they
  don't pass through the deployed service for creation.
- User tokens are long-lived because MCP clients cannot refresh
  tokens. Revocation is the primary access control mechanism.
- The signing key retrieval method depends entirely on the
  deployment tooling. This skill can't prescribe a single command
  — investigate the deployment configuration to trace where the
  key lives.
