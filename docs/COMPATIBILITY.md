# Compatibility

How this repo's structure conforms to skill standards and how major tools
consume it as a source of skills. Verified behaviors documented below.

## How `gemini skills install` Works

The command accepts a git URL (or local path) and scans for skills:

- **Scans one level deep** for directories containing `SKILL.md`
- **Does NOT recurse** into nested subdirectories
- **`--path`** scopes to a subdirectory within the repo before scanning
- **Dotfiles and README.md** are ignored (only `*/SKILL.md` matters)

## Use Cases Tested

### Use Case 1: Single skill at repo root

```
repo/
└── SKILL.md
```

| Command | Result |
|---|---|
| `gemini skills install <url>` | Installs the one skill |

### Use Case 2: Multiple skills at root

```
repo/
├── skill-a/SKILL.md
└── skill-b/SKILL.md
```

| Command | Result |
|---|---|
| `gemini skills install <url>` | Installs skill-a AND skill-b |
| `gemini skills install <url> --path skill-a` | Installs skill-a only |

### Use Case 3: Skills under a subdirectory

```
repo/
├── README.md
└── skills/
    ├── skill-c/SKILL.md
    └── skill-d/SKILL.md
```

| Command | Result |
|---|---|
| `gemini skills install <url>` | Finds nothing (skills are 2 levels deep) |
| `gemini skills install <url> --path skills` | Installs skill-c AND skill-d |
| `gemini skills install <url> --path skills/skill-c` | Installs skill-c only |

### Use Case 4: Mixed — root skills + nested collections

```
repo/
├── skill-1/SKILL.md
├── skill-2/SKILL.md
├── claude/
│   ├── skill-3/SKILL.md
│   └── skill-4/SKILL.md
└── weird-skills/
    └── skill-5/SKILL.md
```

| Command | Result |
|---|---|
| `gemini skills install <url>` | Installs skill-1 and skill-2 only (root level) |
| `gemini skills install <url> --path claude` | Installs skill-3 and skill-4 |
| `gemini skills install <url> --path weird-skills/skill-5` | Installs skill-5 only |
| `gemini skills install <url> --path skill-1` | Installs skill-1 only |

## Skill Name Resolution

The installed skill name comes from the `name` field in SKILL.md frontmatter,
NOT from the directory name. If the directory is `my-folder/` but the SKILL.md
says `name: actual-name`, it installs as `actual-name`.

## Per-Skill Enable/Disable (Gemini)

Gemini supports per-skill enable/disable even within extensions:

```bash
gemini skills disable <name> [--scope user|workspace]
gemini skills enable <name>
```

Disable writes `{"skills": {"disabled": ["name"]}}` to settings.json.
Verified by testing: disabled skill-a within a multi-skill extension, agent
only saw skill-b, re-enabled, both visible again.

## Claude Code

Claude Code has NO skill management CLI. Skills are managed via filesystem:
- Place in `~/.claude/skills/<name>/SKILL.md` (user scope)
- Place in `.claude/skills/<name>/SKILL.md` (project scope)
- `--disable-slash-commands` disables ALL skills for a session (no per-skill control)

## Platform-Specific Frontmatter

Claude-specific frontmatter fields (e.g., `disable-model-invocation`,
`allowed-tools`, `context`) are ignored by Gemini. Verified: installed a skill
with Claude-specific fields to Gemini — installed and listed without error.

## echomodel CLI (em)

`em skills install <name>` provides unified cross-platform installation:
- Resolves skills from registered marketplaces
- Fetches latest from git with 5s timeout, falls back to cache
- Writes to `~/.claude/skills/` (direct) and delegates to `gemini skills install` for Gemini
- `--target claude|gemini` to install to one platform only
