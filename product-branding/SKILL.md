---
name: product-branding
description: >
  Name software products and check availability across domains, GitHub orgs,
  and package registries (PyPI, npm, crates.io, Docker Hub, Homebrew). Use when
  brainstorming product names, validating a name idea, or comparing candidates
  across namespaces.
---

# Product Branding & Name Discovery

Help the user find a strong, available name for a software product. This skill
covers brainstorming names AND checking availability across key namespaces.

## Input Gathering

Start by understanding what needs a name. Accept any combination of:

- **A description or pitch** of the product idea
- **A git repo or GitHub URL** — read the README and key markdown files to
  understand the product
- **Multiple repos** for an ecosystem of related products
- **An existing name** the user wants to validate or find alternatives to

If given a repo path or URL, read `**/*.md` files (especially README.md,
CONTRIBUTING.md, docs/) and scan the codebase structure to understand the
product's purpose, audience, and technical domain.

Ask clarifying questions if needed:
- Target audience (developers, enterprises, end users)?
- Tone (playful, professional, technical, minimal)?
- Constraints (max length, must contain a keyword, must be a real word)?
- Is this part of a family of products that should share naming conventions?

## Name Generation

Generate **at least 5 name candidates** per round. Aim for variety across these
styles:

| Style | Example patterns |
|-------|-----------------|
| **Descriptive** | What it does, literally (CloudSync, DataPipe) |
| **Metaphorical** | Evocative imagery (Lighthouse, Forge, Conduit) |
| **Portmanteau** | Blended words (Kubernetes, Grafana) |
| **Short/punchy** | 1-2 syllables, memorable (Deno, Bun, Rye) |
| **Abstract** | Coined words, no literal meaning (Vercel, Supabase) |
| **Domain-rooted** | References the technical domain (AgentKit, ToolChain) |

For AI/agent ecosystem products specifically, consider:
- References to agency, autonomy, orchestration, reasoning
- Names that work as both a CLI command and a brand
- Names that compose well with subcommands (`<name> deploy`, `<name> run`)
- Whether the name works as a Python/npm package import

## Availability Checks

For each candidate name, check availability across these namespaces. Run checks
in parallel where possible.

### Required Checks

1. **Web domain** — Run `whois <name>.com` (and `.dev`, `.io`, `.ai` if
   relevant). Look for "No match" / "NOT FOUND" / "Domain not found" to
   confirm availability. If taken, note whether it's parked/for-sale vs
   actively used.

2. **GitHub organization** — Run `gh api /users/<name> 2>&1`. A 404 means
   available. If taken, check whether the org is active or dormant.

3. **GitHub repository** — If the user has a preferred org, check
   `gh api /repos/<org>/<name> 2>&1`.

4. **General web presence** — Web search for `"<name>" software` or
   `"<name>" developer tool` to check for existing products with the same
   name, even without matching domains/orgs.

### Conditional Checks (based on product type)

5. **npm registry** — `npm view <name> 2>&1`. "404" means available.
   Also check scoped: `npm view @<org>/<name> 2>&1`.

6. **PyPI** — `pip index versions <name> 2>&1`. "No matching distribution"
   or error means available. Can also web-fetch
   `https://pypi.org/pypi/<name>/json` — 404 means available.

7. **crates.io** — Web-fetch `https://crates.io/api/v1/crates/<name>` —
   look for 404 or empty result.

8. **Docker Hub** — Web-fetch
   `https://hub.docker.com/v2/repositories/library/<name>/` — 404 means
   available (for official images). For user repos, check
   `https://hub.docker.com/v2/repositories/<name>/`.

9. **VS Code Marketplace** — Web search
   `site:marketplace.visualstudio.com "<name>"`.

10. **Homebrew** — `brew info <name> 2>&1`. "No available formula" means
    available.

### Bonus Checks (suggest if relevant)

- **Twitter/X handle** — Web search `site:x.com/<name>` or
  `site:twitter.com/<name>`
- **Subreddit** — Web search `site:reddit.com/r/<name>`
- **Trademark** — Web search `"<name>" site:uspto.gov` (US) for obvious
  conflicts

## Output Format

Present results as a comparison table:

```
## Name Candidates

| # | Name | .com | .dev | GitHub Org | PyPI | npm | Web Conflict |
|---|------|------|------|------------|------|-----|--------------|
| 1 | ...  | ✅   | ❌   | ✅         | ✅   | ❌  | None         |
| 2 | ...  | ❌   | ✅   | ✅         | ✅   | ✅  | Minor        |
| ...                                                              |

✅ = available  ❌ = taken  ⚠️ = parked/dormant/ambiguous
```

After the table, provide a **brief analysis** for each name:
- Pronunciation and memorability
- How it reads as a CLI command or import name
- Any potential confusion with existing products
- Domain alternatives if .com is taken (.dev, .io, .ai, .sh, .run, .tools)

## Iteration

After presenting results, ask if the user wants to:
1. **Explore variations** of a favorite (prefixes, suffixes, alternate TLDs)
2. **Generate more names** with adjusted criteria
3. **Deep-dive** a specific name (full trademark search, social handles, etc.)
4. **Check a name the user thought of** against all namespaces

## Tips for the Agent

- Run namespace checks in parallel using multiple tool calls — don't
  serialize whois/gh/npm/pip checks sequentially.
- `whois` output varies by registrar. Look for any of: "No match",
  "NOT FOUND", "Domain not found", "No Data Found", "AVAILABLE".
  If output contains registration dates or nameservers, the domain is taken.
- `gh api` returns JSON on success (org exists) and error text on 404
  (available). Check the HTTP status, not the body content.
- Some names will be taken everywhere — that's fine, include them if the
  name is strong. Note what's available and what's not.
- For package registries, also check common variations (hyphenated,
  with `-py` or `-js` suffix, prefixed with `py-` or `node-`).
- If a name is taken on GitHub but available as `<name>-dev`, `<name>-hq`,
  `<name>-io`, or `get<name>`, mention that as an alternative.
