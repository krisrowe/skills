---
name: implement-cli-config
description: Design and implement CLI configuration subcommands. Each config option is a named subcommand with its own validation, not a generic key-value store. Covers credential storage strategy (OS keychain vs file), config persistence patterns, and error hinting.
metadata:
  version: "1.0.0"
---

# Implement CLI Config

When the user invokes this skill, they want to add user-facing configuration
to a CLI tool — persistent settings that affect how the tool behaves across
sessions.

## Core Principle: Named Subcommands, Not a Key-Value Store

Every config option is its own subcommand with its own validation, help text,
and allowed values. Never implement config as a generic `set <key> <value>`
grab bag.

**Good:**
```bash
my-tool config signing-key-store os
my-tool config signing-key-store file
my-tool config region us-central1
```

**Bad:**
```bash
my-tool config set signing-key-store os
my-tool config set region us-central1
```

Each subcommand:
- Has its own `--help` with allowed values
- Validates input (e.g., `signing-key-store` only accepts `os` or `file`)
- Can have custom logic (e.g., testing keychain access before accepting `os`)
- Is discoverable via `my-tool config --help`

Show current values:
```bash
my-tool config signing-key-store
# os

my-tool config region
# us-central1
```

No argument = show current value. With argument = set new value. Same pattern
as `git config` but scoped to named subcommands rather than arbitrary keys.

## When to Use `config` vs a Top-Level Command Group

`config` is for **settings that change how the tool behaves** — preferences,
defaults, backend choices. It is not for **problem-domain resources** —
the objects that the tool exists to operate on.

| `config` (tool preferences) | Top-level group (problem-domain resources) |
|-----------------------------|-------------------------------------|
| Output format (json/table) | Servers |
| Color on/off | Users |
| Storage backend (os/file) | Connections |
| Default timeout | Projects |

**Test**: "If I deleted this, would I lose work or just a preference?"

- Lose a preference → `config`
- Lose a managed resource → its own command group

Domain objects get CRUD operations (`add`, `remove`, `list`). Config values
get get/set semantics (no argument = show, with argument = set). Don't mix
them — a user managing servers should never find themselves inside `config`.

## Hierarchical Config

When config has natural groupings, use nested command groups. Each group is
a scope. Properties within a scope are subcommands of that group.

```bash
# Flat (simple tools)
my-tool config region us-central1
my-tool config signing-key-store os

# Hierarchical (tools with scoped config)
my-tool config fleet default work
my-tool config fleet path ops/fleet.yaml

my-tool config auth signing-key-store os
my-tool config auth token-duration 3600

my-tool config display format json
my-tool config display color off
```

Show all values in a scope:
```bash
my-tool config auth
# signing-key-store: os
# token-duration: 3600
```

Show everything:
```bash
my-tool config
# fleet:
#   default: work
#   path: ops/fleet.yaml
# auth:
#   signing-key-store: os
#   token-duration: 3600
# display:
#   format: json
#   color: off
```

### Managed lists

Some config scopes contain a list of items, not just key-value pairs. These
get CRUD subcommands:

```bash
# Example: known servers
my-tool servers add https://api.example.com --name prod
my-tool servers add https://staging.example.com --name staging
my-tool servers list
my-tool servers remove staging
```

Lists are not generic — each list type is its own command group with typed
validation. `servers add` validates a URL and tests connectivity.
`servers remove` validates against known names. These are not
`config set servers.prod.url ...`.

#### Naming CRUD operations

| Verb | When to use |
|------|------------|
| `add` / `remove` | Default. Adding/removing an item from a collection. Prefer `add` over `set` when the user needs to know whether the item already existed — `add` can warn or error on duplicates, while `set` silently overwrites. Use `add` when blind overwrites would be surprising or destructive. |
| `register` | When adding does meaningful work beyond saving — validation, remote verification, setup side effects. Idempotent by nature: re-registering the same item updates it rather than erroring. Similar to `set` but for operations that are heavier than saving a value. |
| `delete` | When removing has meaningful side effects — cleanup, deregistration, remote calls. Use instead of `remove` when it's destructive beyond forgetting the entry. |
| `set` | When there's exactly one value, not a collection. Too light for operations that validate, clone, or verify. |

### When to nest vs keep flat

- **Flat**: fewer than ~5 config values total. Nesting adds ceremony for
  no benefit.
- **Nested**: config values naturally group into 2+ scopes with 2+ values
  each. The grouping should match how users think about the config, not
  how the code is organized.
- **Managed list**: the config is a collection of similar items that grow
  and shrink (fleets, registered servers, known hosts). Always a dedicated
  command group, never nested under `config`.

### Config file structure mirrors the CLI

The on-disk format should match the CLI hierarchy:

```json
{
  "fleet": {
    "default": "work",
    "path": "ops/fleet.yaml"
  },
  "auth": {
    "signing_key_store": "os",
    "token_duration": 3600
  },
  "display": {
    "format": "json",
    "color": false
  }
}
```

CLI path maps directly to JSON path: `my-tool config auth signing-key-store`
reads/writes `auth.signing_key_store`. No translation layer, no surprises.

## Architecture: Core Logic Owns Config, CLI is Thin

All config logic lives in the core application layer — not in the CLI
framework (Click, argparse, etc.). The CLI is a thin wrapper that calls
core functions and formats output. Where the core logic lives depends on
the project: `sdk/`, `core/`, `lib/`, or wherever the central application
logic is. The point is it's not in the CLI layer.

**Core logic owns:**
- Config directory resolution (`get_config_dir()`)
- Loading and saving config files
- Validation rules (allowed values, type checks, required fields)
- Keychain read/write via `keyring`
- Default values and resolution order (flag → env → config → error)

**CLI owns:**
- Parsing arguments and flags
- Calling core functions with parsed values
- Formatting output for the terminal
- Showing error messages with hints

```python
# Core — my_app/config.py (or sdk/config.py, lib/config.py, etc.)
def get_signing_key_store() -> str:
    """Return current signing key storage backend."""
    return load_config().get("auth", {}).get("signing_key_store", "os")

def set_signing_key_store(value: str) -> None:
    """Set signing key storage backend. Validates input."""
    if value not in ("os", "file"):
        raise ValueError(f"Invalid backend: {value}. Must be 'os' or 'file'.")
    if value == "os":
        _test_keychain_access()
    config = load_config()
    config.setdefault("auth", {})["signing_key_store"] = value
    save_config(config)

# CLI — my_app/cli.py (thin wrapper)
@config_group.command("signing-key-store")
@click.argument("value", required=False, type=click.Choice(["os", "file"]))
def signing_key_store(value):
    """Get or set where signing keys are stored."""
    from my_app.config import get_signing_key_store, set_signing_key_store
    if value is None:
        click.echo(get_signing_key_store())
        return
    set_signing_key_store(value)
    click.echo(f"Signing key store set to: {value}")
```

This ensures:
- All interfaces (CLI, MCP tools, API) use the same config logic
- Config behavior is testable without invoking the CLI framework
- Validation rules are consistent regardless of entry point
- A future programmatic consumer gets the same behavior

## Credential Storage Strategy

When a CLI needs to persist secrets (API keys, signing keys, tokens):

### OS keychain as default

Use the `keyring` Python library for OS-native credential storage:

```python
import keyring

keyring.set_password("my-tool", "signing-key", value)
value = keyring.get_password("my-tool", "signing-key")
```

- **macOS**: Keychain (zero config, always available)
- **Windows**: Credential Manager (zero config, always available)
- **Linux desktop**: GNOME Keyring / KWallet (available on desktop installs)

### When keychain fails

Corporate machines may block keychain access. Headless Linux has no keychain
daemon. CI environments have neither.

**Do not silently fall back.** A silent fallback means the user doesn't know
where their secret is stored — it becomes unpredictable and undebuggable.

Instead: fail with an actionable hint.

```
Error: OS keychain unavailable.
Run: my-tool config signing-key-store file
```

The user explicitly chooses their storage backend once. Every subsequent
command uses that choice. No ambiguity.

### Storage backends

| Backend | Config value | Where secrets go | When to use |
|---------|-------------|-----------------|-------------|
| `os` | Default | OS keychain via `keyring` | Desktop machines |
| `file` | Explicit | `~/.config/my-tool/` | Corp lockdown, headless, preference |

The `file` backend should warn on first use that secrets will be stored on
disk. Consider file permissions (`0600`) but do not implement encryption
unless the project specifically requires it — encrypted-at-rest with a key
derived from... what? It's turtles all the way down. Be honest about the
trade-off.

### What doesn't go in the keychain

Only secrets go in the keychain. Non-secret config (URLs, regions, feature
flags) goes in the config file. Never mix them.

| Value | Where |
|-------|-------|
| Signing key | Keychain or file (per `signing-key-store` setting) |
| API token | Keychain or file |
| Base URL | Config file always |
| Region | Config file always |
| Feature flags | Config file always |

## Config File Location

Follow XDG on all platforms:

```python
import os
from pathlib import Path

def config_dir() -> Path:
    xdg = os.environ.get("XDG_CONFIG_HOME", os.path.expanduser("~/.config"))
    return Path(xdg) / "my-tool"
```

Support env var override for testing:

```python
def config_dir() -> Path:
    override = os.environ.get("MY_TOOL_CONFIG_DIR")
    if override:
        return Path(override)
    xdg = os.environ.get("XDG_CONFIG_HOME", os.path.expanduser("~/.config"))
    return Path(xdg) / "my-tool"
```

Tests use `monkeypatch.setenv("MY_TOOL_CONFIG_DIR", str(tmp_path))` to
isolate from the real filesystem.

## Config Resolution Order

For values that can come from multiple sources:

1. **Explicit flag** (`--url`, `--signing-key`) — always wins
2. **Environment variable** (`MCP_APP_URL`) — session-scoped
3. **Config file** (`~/.config/my-tool/config.json`) — persistent
4. **Error** with hint about how to set it

This order is not a fallback chain — it's a precedence. If a flag is
provided, skip everything else. If an env var is set, skip config file.
Never silently merge values from multiple sources.

## Implementation Pattern

```python
import click

@click.group()
def config():
    """View and set configuration."""
    pass

@config.command("signing-key-store")
@click.argument("value", required=False, type=click.Choice(["os", "file"]))
def signing_key_store(value):
    """Get or set where signing keys are stored.

    OS uses the operating system keychain (macOS Keychain, Windows
    Credential Manager, Linux Secret Service). FILE stores keys in
    the config directory.
    """
    if value is None:
        current = load_config().get("signing_key_store", "os")
        click.echo(current)
        return

    if value == "os":
        # Test keychain access before accepting
        try:
            import keyring
            keyring.get_password("my-tool", "__test__")
        except Exception:
            raise click.ClickException(
                "OS keychain is not available. Use 'file' instead."
            )

    save_config_value("signing_key_store", value)
    click.echo(f"Signing key store set to: {value}")
```

## Error Messages

Every error should tell the user exactly what to do next:

```
Error: No base URL configured.
Run: my-tool set-base-url <url>
Or set MY_TOOL_URL environment variable.
```

```
Error: OS keychain unavailable.
Run: my-tool config signing-key-store file
```

```
Error: No signing key.
Run: my-tool set-base-url <url> --signing-key <key>
Or set MY_TOOL_SIGNING_KEY environment variable.
```

Never show a generic error without a next step. The user should be able to
copy-paste from the error message to fix the problem.

## Testing

Config tests must be isolated from the real filesystem and real keychain.

```python
@pytest.fixture(autouse=True)
def isolate_config(tmp_path, monkeypatch):
    monkeypatch.setenv("MY_TOOL_CONFIG_DIR", str(tmp_path / "config"))
```

For keychain tests, mock `keyring.get_password` and `keyring.set_password`
at the system boundary. Do not mock the config layer — test it against
real files in tmp_path.
