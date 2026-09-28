# Short Technical Writing

A skill for AI coding agents that rewrites code comments, PR descriptions and docs into Simplified Technical English, then cuts their length by a target percentage. It runs two passes in order: first the plain-English rewrite, then a measured cut. It keeps the facts a reader needs and deletes the lines that restate the code.

## Install

### Claude Code

```bash
mkdir -p ~/.claude/skills/short-technical-writing
cp SKILL.md ~/.claude/skills/short-technical-writing/SKILL.md
```

### Other AI agents

`SKILL.md` is plain markdown. Point your agent at it or paste it into a system prompt.

## Usage

```
/short-technical-writing
```

Point it at a diff's comments, a PR description or a document. Give a target cut, or use the default.
