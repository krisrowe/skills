---
name: write-verified-advisory
description: Author professional, evidence-based technical advisories, consultant reports, or research memos as local .md files. This skill is invoked when asked to summarize technical research, provide formal guidance to stakeholders, or document empirical findings from a session. Triggers include phrases like "write a report," "prepare an advisory," "summary for the client," or "technical memo."
effective_targets: ['claude', 'gemini']
---

# Verified Advisory & Technical Report Creation

This skill defines a standardized workflow for producing authoritative, research-backed technical reports. It mandates a dual-output structure: a polished stakeholder report and a raw verification audit for the author.

## Mandates

### 1. Hallucination Defense: Multi-Tiered Verification
Every technical claim regarding product capabilities, standards, or lifecycles MUST be backed by a live, verified source. Do NOT rely on internal training data for specific facts.

**Verification Hierarchy (Highest to Lowest Fidelity):**
1.  **Browser Automation (Optional/If Available):** Use tools for browser automation (e.g., Playwright, Puppeteer, or an MCP browser tool) to render the page and confirm the claim. Preferred for JS-heavy documentation sites.
2.  **Deterministic Primitives (Mandatory Fallback):** Use `curl -L -s` to retrieve raw page content. Inspect the raw response body to confirm the exact presence of the claim or quote.
3.  **Prohibited:** NEVER cite a source based solely on a high-level search summary or an agent's "understanding" of a page without inspecting the raw source content or a rendered screenshot/text dump.

### 2. Dual-Report Output
The agent MUST produce two distinct reports for every invocation:

#### Report A: The Polished Advisory (Stakeholder-Facing)
- **Format:** Saved as a local `.md` file.
- **Content:** Professional, narrative, and authoritative.
- **Verification:** Woven into the text using hypertext and quotes. No footnotes or raw audit data.
- **Creative Examples:**
    - "As confirmed by [Apigee's official documentation](URL), the policy is strictly bound to its own local bundle..."
    - "This behavior is consistent with the [spec's runtime requirements](URL), where we find the following quote: '...'"

#### Report B: The Verification Audit & Gap Analysis (Author-Facing)
- **Format:** Rendered as a response in the chat session.
- **Content:** Raw, thorough, and technical.
- **Audit Trail:** Lists every fact/claim from Report A and provides:
    - The exact command used to verify (e.g., `curl -L -s <URL> | grep '...'`).
    - The timestamp of the verification.
    - A snippet of the raw response that confirmed the claim.
- **Gap Analysis:** Lists any claims that were not fully sourced, explains why certain items were categorized as assumptions, and identifies "misses" for potential iteration.

### 3. Structure & Tone
- **Tone (Report A):** Professional, direct, and authoritative (Senior Consultant/Engineer persona).
- **Required Sections (Report A):**
    - **Executive Summary / Context:** Concise statement of the issue or request.
    - **Verified Technical Findings:** Detailed evidence with verified links and quotes.
    - **Advisory & Recommendations:** Clear, actionable guidance derived from the findings.
    - **Caveats:** Only include critical constraints for the stakeholder (e.g., version limits).

### 4. Privacy & Portability
- **Anonymity:** NEVER include customer names, specific organization identifiers, or the user's employer details.
- **Path Neutrality:** Never include local absolute paths or machine-specific locations. Use generic placeholders like `<PROJECT_ROOT>` or `<CLIENT_ORG>`.
- **No Signatures:** Avoid personal signatures or platform-specific metadata.
- **No Dependencies:** Rely on standard primitives (curl, grep, etc.) as the baseline.

## Workflow

1.  **Drafting:** Outline the advisory based on session research.
2.  **Verification:** Execute the verification hierarchy for every assertion.
3.  **Synthesis:** Construct **Report A** (.md file) with polished citations.
4.  **Audit:** Construct **Report B** (chat response) with the full verification audit trail.
5.  **Delivery:** Provide the local file path for Report A and display Report B in the chat.
