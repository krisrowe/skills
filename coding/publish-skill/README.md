# publish-skill

Built a skill? This one walks your agent through publishing it — finding the right marketplace repo, validating the SKILL.md, placing it in the correct collection, and pushing.

Handles the mundane parts so you focus on the skill content, not the publishing mechanics.

Cross-platform — works with any AI coding agent that supports the SKILL.md format.

## Install

```bash
pipx install echomodel
em skills install publish-skill
```

Or install directly with Gemini CLI:

```bash
gemini skills install https://github.com/echo-skill/echoskill.git --path coding/publish-skill
```

See the [echoskill README](../../README.md#quick-start) for full setup.
