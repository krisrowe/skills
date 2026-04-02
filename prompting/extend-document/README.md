# extend-document

Safely add content to an existing document without losing what's already there. Your agent follows a strict bidirectional iteration protocol — checking that nothing from the original was lost AND that everything from the source was captured — with commits between each step so you can always roll back.

Works with or without a git repo. In a repo, uses commits and `git diff`. Without one, backs up to a temp directory and uses `diff` for the same guarantees.

Cross-platform — works with any AI agent that supports the SKILL.md format.

## Install

```bash
pipx install echomodel
em skills install extend-document
```

