---
name: extend-document
description: >-
  Safely extend or integrate content into an existing document without losing
  original content. Use when appending research findings, session notes, or
  new sections to an existing .md file. Ensures bidirectional integrity through
  iterative commits and diffs.
user-invocable: true
---

# Extend Document

Safely integrate new content into an existing document while guaranteeing no
original content is lost and all source material is fully captured.

## Prerequisites

Determine the safety mechanism before making any edits:

**If the target file is in a git repo:**
- Ensure the working tree is clean (no uncommitted changes to the target file)
- If dirty, commit or stash first — never start edits on an uncommitted file
- Use git commits between each step and `git diff` for comparisons

**If the target file is NOT in a git repo:**
- Copy the file to a temp directory as a backup before the first edit
- Use `diff` (or `git diff --no-index` if git is installed) for comparisons
- Keep the backup until all iterations complete successfully

## Step-by-step procedure

Follow these steps in order. Do not skip or combine steps.

### Step 1: Establish baseline

- Read the target document in full
- If in a git repo: confirm clean state with `git status`
- If not in a git repo: copy to temp backup (`cp target.md /tmp/target-backup-$(date +%s).md`)
- Record the baseline state (commit SHA or backup path)

### Step 2: Draft the integration

- Read the source material (session context, research notes, etc.)
- Plan where new content fits in the existing document structure
- Prefer appending new sections or extending existing sections over rewriting
- Preserve all original content — do not rephrase, summarize, or reorganize
  existing text unless explicitly asked

### Step 3: Apply edits

- Make the edits to the target document
- If in a git repo: commit immediately with a descriptive message
- If not: save the file

### Step 4: Forward check — was anything lost from the original?

- Diff the current version against the baseline (Step 1)
  - Git: `git diff <baseline-sha> HEAD -- target.md`
  - No repo: `diff /tmp/target-backup.md target.md`
- Review every deletion and modification in the diff
- If any original content was lost or altered unintentionally:
  - Restore it
  - Commit (or save)
  - Repeat this step until the diff shows only additions and intentional edits

### Step 5: Reverse check — was everything from the source captured?

- Re-read the source material (session context, notes, etc.)
- Compare against what was added to the document
- For each piece of valuable information in the source:
  - Is it present in the document?
  - Is it in the right section?
  - Is it complete (not truncated or summarized)?
- If anything is missing:
  - Add it
  - Commit (or save)
  - Repeat this step until all source material is captured

### Step 6: Iterate until stable

- Repeat Steps 4 and 5 alternately
- Continue until **both directions** produce no changes for at least two
  consecutive iterations
- Each iteration must be committed (or saved) separately

### Step 7: Final verification

- Diff the final version against the original baseline from Step 1
  - Git: `git diff <baseline-sha> HEAD -- target.md`
  - No repo: `diff /tmp/target-backup.md target.md`
- Confirm:
  - Every line removed from the original was intentional
  - Every piece of source material is present
  - No unintended reformatting, reordering, or summarization occurred
- Report confidence level and summary of changes

### Step 8: Cleanup

- If not in a git repo: inform the user where the backup is, suggest keeping
  it until they've reviewed the result
- If in a git repo: the commit history serves as the backup

## Rules

- **Never start on a dirty file.** Always commit or backup first.
- **Commit after every substantive change.** Small commits make diffs readable.
- **Preserve original voice.** Don't rewrite existing content to match new content's style.
- **Additions over modifications.** Prefer adding new sections over editing existing ones.
- **Report what changed.** After completion, summarize: what was added, what (if anything) was modified, and confirm nothing was lost.
