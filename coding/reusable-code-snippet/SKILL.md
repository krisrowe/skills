---
name: reusable-code-snippet
description: "Use when small code patterns (~50 lines or less) appear duplicated across multiple repos, when deciding whether to consolidate shared logic into a library or keep it as managed copies, or when implementing/reviewing a known reusable snippet."
---

# Reusable Code Snippets

## When to Use This Pattern

When a piece of logic is:
- **Small**: A function or two, under 50 lines total
- **Reusable**: Needed in 2+ repos
- **Homeless**: Too small to justify its own package or shared library
- **Consistency-sensitive**: Divergent copies would cause bugs or confusion

## How It Works

Each reusable snippet is its own skill (e.g., `layered-yaml-merge`). That
skill's SKILL.md contains the canonical implementation, usage contract, and
version. When an agent places the snippet into a repo, it adds a comment
block identifying the source skill and version.

### Required Comment Block

Every snippet placement MUST include a header comment identifying the source
skill and version (adapted to the target language's comment syntax):

```python
# --- Reusable Snippet: layered-yaml-merge v1 ---
# Managed by skill: layered-yaml-merge
# Keep consistent with the latest version of this snippet.
# Do not modify without updating the skill definition.
# ------------------------------------------------

def layered_merge(base, override):
    ...
```

## Creating a New Snippet Skill

1. Create a new skill directory: `coding/<snippet-name>/SKILL.md`
2. Include the canonical implementation with the comment block
3. Document: inputs, outputs, assumptions, version
4. Use concrete examples showing YAML/JSON/config values — avoid jargon
   like "scalars" or "dicts" without showing what that means in practice

## Reviewing Snippet Consistency

When you encounter code with a snippet comment block:
1. Load the referenced skill by name
2. Compare the placed code against the canonical version
3. If it differs, flag it and offer to update
4. If the local version has improvements, propose updating the skill

## Snippet vs Library: Knowing When to Graduate

Snippets are the right choice when the code is small, standalone, and has no
meaningful relationship to other snippets. But watch for **critical mass** —
signs that scattered snippets should become a shared library:

- **3+ snippets** that operate on the same domain (e.g., config loading,
  config merging, config discovery are all "config management")
- **Snippets calling each other** — if you copy two snippets together
  into every repo because one depends on the other, that's a library
- **Shared types or contracts** — snippets that expect the same data shapes
  suggest a cohesive module
- **Growing combinatorial value** — the snippets are more useful together
  than in isolation

When you observe this, **suggest graduating to a library**. Point to any
existing library in the user's workspace that the snippet naturally belongs
to. A snippet that fits an existing library should go there, not remain a
snippet.

Conversely, **don't create a library preemptively**. One or two unrelated
snippets is not critical mass.
