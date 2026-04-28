# claude-skills

A collection of Claude Code skills for software-engineering and project-hygiene tasks.

Each skill lives in its own directory with a `SKILL.md` and any supporting files. Claude loads a skill when the user's request matches its trigger description, then follows the procedure in `SKILL.md`.

## Install

Skills install by being placed under `~/.claude/skills/`. Clone this repo there:

```bash
git clone https://github.com/sipaan/claude-skills.git ~/.claude/skills/claude-skills
```

Or symlink an individual skill:

```bash
git clone https://github.com/sipaan/claude-skills.git
ln -s "$PWD/claude-skills/doc-rot-audit" ~/.claude/skills/doc-rot-audit
```

Either approach makes the skill discoverable to Claude Code on your next session.

## Skills in this repo

- **[doc-rot-audit](doc-rot-audit/)** — Audit project documentation against the actual codebase to find and report doc rot: fictional content, aspirational content presented as fact, stale claims from past refactors, and inventory drift. Stack-agnostic. Produces a structured report with file:line evidence; does not auto-fix.

## License

[MIT](LICENSE)
