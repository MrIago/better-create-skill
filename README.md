# /better-create-skill 🛠️

**The best Claude Code skill creator — build a skill end to end, from a vague
idea to a published, installable plugin.**

The official `skill-creator` is great at *process* (draft → eval → iterate). This
one adds everything you only learn by actually shipping a skill: the **mechanism**
(`${CLAUDE_SKILL_DIR}`, context lifecycle, the full frontmatter), **when and how
to bundle scripts**, **multi-OS portability**, **user API-key config**, and
**publishing to a GitHub plugin marketplace** so anyone can install it.

```
/better-create-skill make a skill that turns my meeting recordings into action items
```

Claude then walks the whole lifecycle — capture intent → decide shape → write
SKILL.md → make it portable & configurable → test → publish — reading the right
reference for each step.

## Why it exists

I built a real, non-trivial skill (a 5-platform content consumer published to a
marketplace) and hit every wall the docs and the official creator don't cover.
This skill is that hard-won knowledge condensed into copy-paste snippets and
rules, organized by the step you're on. It's **knowledge, not a generator** —
every skill is different, but the patterns repeat.

## What's inside

`SKILL.md` is the orchestrator; the depth lives in `references/`, loaded on demand:

| Reference | Covers |
|-----------|--------|
| `anatomy.md` | layout, progressive disclosure, what goes where |
| `frontmatter.md` | every field, command naming, `${CLAUDE_SKILL_DIR}`, `$ARGUMENTS`, `!` injection, validation rules |
| `scripts-pattern.md` | when to bundle a script, calling it, duplicate-then-DRY, parallelism, batch entry points |
| `multi-os.md` | `python`/`python3`, paths, GPU fallback, OS-only deps, preflight check |
| `secrets-and-config.md` | env-var-OR-`.env` pattern, configurable via chat, graceful backend cascade |
| `testing.md` | validating + iterating without overfitting |
| `publishing.md` | plugin.json, marketplace.json, repo layouts, GitHub publish, install/update cycle |

## Install

```
/plugin marketplace add MrIago/better-create-skill
/plugin install better-create-skill@better-create-skill
```

Or manually:

```bash
git clone https://github.com/MrIago/better-create-skill.git ~/.claude/skills/better-create-skill
```

Then: `/better-create-skill <describe the skill you want>`.

## License

MIT © [Iago Lima Toledo](https://github.com/MrIago)
