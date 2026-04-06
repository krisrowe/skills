# Contributing

## Repo Structure

The structure of this repo is deliberate and must be maintained for
compatibility with `gemini skills install` and `em skills install`.

- Each skill is a directory containing a `SKILL.md` file
- Skills are organized into **collections** (folders)
- `gemini skills install <url>` scans one level deep for `*/SKILL.md`
- `gemini skills install <url> --path <collection>` scans within that collection
- `gemini skills install <url> --path <collection>/<skill>` installs one skill

See the [compatibility guide](docs/COMPATIBILITY.md) for how this repo's
structure works with major skill consumers across platforms.

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

## Tool Dependencies

Skills in a general-purpose marketplace must not hard-depend on tools that
the average consumer of an open marketplace of skills is unlikely to have
installed. If a skill needs a non-ubiquitous tool, it must offer 3+
alternatives so it works for everyone. See the
[develop-skill](coding/develop-skill/SKILL.md) for the full pattern and
ordering rules.

## Skill Memories

Skills can instruct the agent to save memories (persistent notes for future
sessions). The agentskills.io specification explicitly permits this:

> "The Markdown body after the frontmatter contains the skill instructions.
> There are no format restrictions. Write whatever helps agents perform the
> task effectively."
>
> — [agentskills.io/specification](https://agentskills.io/specification),
> "Body content" section

Since skills are loaded as instructions into the agent's context, and both
Claude Code and Gemini CLI agents have memory tools available during normal
operation, a skill instruction like "save a memory recording which method
was used" triggers normal tool use — no special mechanism needed.

### Guidelines for skill authors

- Be explicit: "After completing this workflow, save a memory noting: the
  publishing method used was [method], the target repo was [repo]."
- Specific instructions produce more reliable results than vague ones.
- Memory instructions are best-effort — agents follow them with high
  reliability but they are instructions, not enforced configuration.
- Use memories for workflow state that speeds up future invocations
  (e.g., which CLI tool is available, which marketplace repo is preferred).
  Do not use memories for information derivable from code or git history.

## `.marketplace` Files

`.marketplace` files are optional dotfiles read by `em` for marketplace
metadata (ratings, quality scores, summaries). They are ignored by
`gemini skills install` and Claude Code. Do not remove them.
