# Setup — html-system

**No credentials or wiring required.** The skill is fully self-contained.

The only thing to know: it ships with two files that must sit together in
`~/.claude/skills/html-system/`:

- `SKILL.md` — the skill instructions
- `template.html` — the build scaffold the skill starts from

The install commands in the [README](README.md) place both correctly. With them
in place, prompting Claude to build a system diagram reproduces the author's
exact behaviour — no API keys, hooks, or other machine-specific setup.
