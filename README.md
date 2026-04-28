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

## Recommended cadence for doc-rot-audit

Doc rot is invisible until someone audits for it. Catching it early is much cheaper than catching it late, when stale docs have already misled new contributors or anchored decisions on wrong assumptions.

**Default cadence: weekly.** Set a calendar reminder (Friday afternoon or Monday morning works well). When it fires, open a Claude Code session in the project root and simply say:

> audit docs for rot

The skill triggers automatically and produces a report you can scan in minutes. Acting on the findings is optional — the value of the audit is even higher when you decide *not* to fix something, because you've made an informed choice to accept the drift.

**One-off triggers — run the audit immediately when:**

- You've just finished a significant refactor. Code change → cascade rot. Catching what shifted now beats discovering it three months later.
- You're returning to a project after a long pause (multi-week or longer). Long-dormant projects accumulate the most rot.
- You're about to onboard a new contributor. They will read the docs cold and trip over every piece of rot. Fix what you can before they start.

For stable projects in maintenance mode, monthly is reasonable instead of weekly.

## License

[MIT](LICENSE)
