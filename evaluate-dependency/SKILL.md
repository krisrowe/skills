---
name: evaluate-dependency
description: Evaluate whether a library, framework, or dependency is solid before adopting it. Use when the user asks "is this library legit?", "should I use X or Y?", "is this maintained?", "how popular is X?", "is X fly-by-night?", "what do people use for HTTP clients in Python?", or any variant of "help me decide whether to depend on this."
metadata:
  version: "1.0.0"
---

# Evaluate Dependency

When the user invokes this skill, they are asking: "Should I trust this library
enough to depend on it?" or "What should I use for X?"

The user does not want to spend brain energy formulating what makes a dependency
trustworthy. This skill codifies that evaluation so the user just names the
library (or the problem space) and gets a structured, data-driven answer.

## What to evaluate

For every candidate library, research and present these dimensions. Use
**concrete numbers**, not vibes. Cite sources.

### 1. Adoption scale

| Metric | Where to find it |
|--------|-----------------|
| Monthly PyPI downloads | pypistats.org, libraries.io, or `pip` stats |
| GitHub stars | GitHub repo page |
| Dependent repos/packages | GitHub "Used by" or libraries.io |
| npm/crates/etc. downloads | Equivalent registries for non-Python |

Context matters more than raw numbers. 500M downloads/month means nothing
without comparison. Always show the candidate alongside its closest
alternatives in a comparison table.

### 2. Open source health

This is non-negotiable context. The user needs to know they can fork it, read
every line, and never be locked in.

- **License** — what is it? (MIT, Apache 2.0, BSD, GPL, proprietary?) Is it
  a permissive license the user can depend on without legal concern?
- **Full source on GitHub** — is the complete, buildable source code in a
  public repo on github.com? Not a mirror, not a read-only export, not a
  "source-available" subset. A real repo you could fork and maintain if the
  maintainer disappeared.
- **Activity** — recent commit frequency, open PRs, open issues, issue
  response time. A repo with 10K stars but no commits in 6 months is a
  different proposition than one with weekly releases.
- **Community** — number of contributors, whether external PRs get merged,
  whether there's meaningful discussion on issues (not just the maintainer
  closing stale tickets).

### 3. Who maintains it

- **Primary maintainer(s)** — name them. Individual or organization?
- **What else have they built?** — list other well-known projects by the same
  maintainer/org. The user will recognize names like Django REST Framework,
  FastAPI, Starlette, pytest. If the maintainer built something the user
  already depends on, that's signal.
- **Notable contributors or backers** — is the library maintained or funded by
  a recognizable company? (e.g., Anthropic, Google, Meta, Encode, Pallets)
- **Bus factor** — is it one person, a small team, or a large org?

### 4. Who depends on it

This is often more telling than download counts. List recognizable projects
and SDKs that have this library as a hard dependency:

- Major framework dependencies (e.g., "Starlette's TestClient subclasses this")
- Major SDK dependencies (e.g., "OpenAI, Anthropic, and Google GenAI SDKs all hard-depend on this")
- If the library is a transitive dependency of something the user's project
  already depends on, say so explicitly — it's already in their tree.

### 5. Maturity and trajectory

- **Age** — when was the first release?
- **Release cadence** — how often are new versions published?
- **Last release date** — is it actively maintained or abandoned?
- **Adoption trajectory** — is it growing, stable, or declining? Compare
  download trends over the past 12-18 months if available.
- **API stability** — has there been a recent major version bump? Are there
  known breaking changes planned?

### 6. Ecosystem position

- **Is it stdlib or stdlib-adjacent?** — Part of the standard library? A
  near-universal dependency that ships with major frameworks?
- **Is it the incumbent, the challenger, or the newcomer?** — Every problem
  space has a landscape. Name where this library sits.
- **What would I use instead?** — Always name the top 2-3 alternatives, even
  if the candidate is the clear winner. The user wants to know the landscape,
  not just the recommendation.

### 7. Fit for the user's context

- **Does it solve the actual problem?** — Sometimes the user is evaluating a
  library that's adjacent to what they need.
- **Is it already in the dependency tree?** — Check the project's current
  dependencies (pyproject.toml, requirements.txt, package.json). If it's
  already a transitive dep, adopting it explicitly costs nothing.
- **Does it add heavy transitive dependencies?** — A library that pulls in
  50 packages is a different proposition than one with zero deps.

## How to present

**Always use a comparison table** for the top candidates in the space, even if
the user only asked about one library. The table grounds the evaluation.

```
| Metric              | httpx          | requests       | aiohttp        |
|---------------------|----------------|----------------|----------------|
| Monthly downloads   | 499M           | 1,264M         | 412M           |
| GitHub stars        | 15.1K          | 53.9K          | —              |
| License             | BSD-3          | Apache 2.0     | Apache 2.0     |
| Full source on GH   | Yes (encode/httpx) | Yes (psf/requests) | Yes (aio-libs/aiohttp) |
| Recent commits      | Weekly         | Monthly        | Weekly         |
| Contributors        | 200+           | 600+           | 400+           |
| Maintainer          | Encode (T.C.)   | PSF (K.R.)     | aio-libs       |
| Also built          | DRF, Starlette | —              | —              |
| Backed by           | Anthropic, OpenAI (as dependents) | —   | —              |
| First release       | 2019           | 2011           | 2014           |
| Async support       | Yes            | No             | Yes            |
| Already in dep tree | Yes            | No             | No             |
```

After the table, give a **direct recommendation** with reasoning. Don't hedge.
If the answer is "use X", say so and say why.

## What this skill is NOT

- **Not a rubber stamp.** If the library is sketchy, say so. Low bus factor,
  stale releases, declining downloads — these are real risks.
- **Not exhaustive research.** The user wants a decision, not a doctoral
  thesis. Hit the key dimensions, present the data, make the call.
- **Not language-specific.** The dimensions above apply to any package
  ecosystem (PyPI, npm, crates.io, Go modules, etc.). Adapt the specific
  lookup methods to the ecosystem.

## When the user asks about a problem space, not a specific library

If the user says "what should I use for HTTP in Python?" rather than "is httpx
any good?", start by mapping the landscape (what are the options?), then
evaluate each candidate using the framework above. End with a recommendation.

## When the user is comparing two specific libraries

Skip the landscape mapping. Go straight to the comparison table. Evaluate both
on all dimensions. Recommend one with clear reasoning.
