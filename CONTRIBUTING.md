# Contributing

## Repo Structure

The structure of this repo is deliberate and must be maintained for
compatibility with `gemini skills install` and `aicfg skills install`.

- Each skill is a directory containing a `SKILL.md` file
- Skills are organized into **collections** (folders)
- `gemini skills install <url>` scans one level deep for `*/SKILL.md`
- `gemini skills install <url> --path <collection>` scans within that collection
- `gemini skills install <url> --path <collection>/<skill>` installs one skill

See [docs/TESTING.md](docs/TESTING.md) for detailed behavior documentation
of how skill installation works across platforms.

## Adding a Skill

1. Create `<collection>/<skill-name>/SKILL.md`
2. Include `name` and `description` in YAML frontmatter (required)
3. Keep the `name` field matching the directory name
4. Write platform-neutral instructions when possible
5. If the skill is platform-specific, place it under `claude/` or `gemini/`

## Frontmatter

Use standard [agentskills.io](https://agentskills.io/specification) fields.
Claude-specific fields (e.g., `disable-model-invocation`) are allowed — Gemini
ignores them.

## No Scripts

Skills in this repo are pure instructions. If a skill requires deterministic
script execution, it belongs in a plugin (Claude) or extension (Gemini).

## `.marketplace` Files

`.marketplace` files are optional dotfiles read by `aicfg` for marketplace
metadata (name, namespace). They are ignored by `gemini skills install` and
Claude Code. Do not remove them.
