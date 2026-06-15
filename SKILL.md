---
name: better-create-skill
description: Create production-grade Claude Code skills end to end — from intent to a published, installable plugin. Use this whenever the user wants to build, create, scaffold, design, or publish a skill, slash command, or Claude Code plugin, even if they just describe a repeatable workflow they keep re-explaining ("turn this into a skill", "make a /command for X", "package this as a plugin"). Covers the full lifecycle the official skill-creator skips — the actual mechanism (SKILL.md body, the skill-dir variable, context lifecycle), bundling scripts, multi-OS portability, secret and key config, and publishing to a GitHub plugin marketplace.
allowed-tools: Bash, Read, Write, Edit, WebFetch
---

# better-create-skill — build skills like a pro

You already know how to write prose. This skill makes you good at the *craft* of
Claude Code skills: the parts that aren't obvious until you've shipped one — how
the mechanism actually works, when to bundle a script, how to make it run on
someone else's machine, how to handle their API keys, and how to publish it so
anyone can `/plugin install` it.

It is **knowledge, not automation** — every skill is different, so there's no
generic generator. Instead each reference file carries copy-paste snippets and
hard-won rules. Read the reference for the step you're on.

## The mindset (read this first)

1. **A skill is progressive disclosure.** Three levels: the `description`
   (always in context, ~100 words), the SKILL.md body (loaded when triggered,
   keep <500 lines), and bundled files/scripts (loaded/run only on demand).
   Design so the cheap level answers most cases. See `references/anatomy.md`.

2. **Fetch on demand, don't pre-compute.** The strongest skills do the minimum
   and only reach for more when a real need (the user's task or question)
   requires it — never speculatively. Bake this principle into the SKILL.md
   instructions themselves.

3. **The body is a recurring token cost.** Once a skill loads, its body stays in
   context for the session. State what to do, not why at length. Move depth to
   `references/`.

4. **Explain the why, don't pile on MUSTs.** Today's models follow reasoning
   better than rigid ALL-CAPS rules. If you're writing NEVER/ALWAYS, reframe as
   "do X because Y".

## The lifecycle — work the step you're on

### 1. Capture intent
What should it enable? When should it trigger (real phrases the user would
type)? What's the output? Is it knowledge (reference) or a task (action)? If the
current conversation already contains the workflow, extract it from there first.

### 2. Decide the shape
- Pure instructions/knowledge → SKILL.md only.
- Repeatable deterministic work (parsing, downloading, transforming, calling an
  API the same way every time) → **bundle a script** and have the skill call it.
  Rule of thumb: if every invocation would otherwise make the model rewrite the
  same helper, bundle it. See `references/scripts-pattern.md`.
- Multiple domains/platforms/variants → one SKILL.md that routes + a
  `references/<variant>.md` each. The model reads only the relevant one.

### 3. Write SKILL.md + frontmatter
Frontmatter rules, the full field list, command-name behavior, and string
substitutions (`${CLAUDE_SKILL_DIR}`, `$ARGUMENTS`) are in
`references/frontmatter.md`. Make the `description` slightly "pushy" — models
under-trigger skills; spell out trigger phrases.

### 4. If it bundles scripts, make them portable & configurable
- **Multi-OS** (`python` vs `python3`, temp paths, GPU/no-GPU, Linux-only deps):
  `references/multi-os.md`.
- **Secrets / API keys** the user must provide (so it works on *their* machine,
  configurable by just telling Claude the key): `references/secrets-and-config.md`.

### 5. Test it
Validate the frontmatter, then trigger the skill with realistic prompts (and a
baseline without it). Process and iteration tips: `references/testing.md`.

### 6. Ship it — personal plain skill (DEFAULT) or plugin (for sharing)

**Default for your own skills: a plain skill, version-controlled, NOT a plugin.**
A skill is just a folder with `SKILL.md` (+ optional `scripts/`, `references/`).
No `.claude-plugin/`, no marketplace, no namespacing (`/name`, not `/plugin:name`).
The workflow keeps a git repo as the source of truth and a clone where Claude
loads it:

```bash
# 1. The skill lives as its own git repo in your dev home:
#    ~/Documentos/Projetos/Pessoal/skills/<name>/   (SKILL.md at the root)
cd ~/Documentos/Projetos/Pessoal/skills/<name>
git init && git add -A && git commit -m "<name> skill"
# 2. Create the GitHub repo + push (so it's backed up + cloneable anywhere):
#    gh repo create MrIago/<name> --public --source=. --push   (or the API)
# 3. Clone it where Claude Code discovers personal skills:
git clone https://github.com/MrIago/<name>.git ~/.claude/skills/<name>
```

**Updating later:** edit in `Pessoal/skills/<name>` → commit → push → then
`git -C ~/.claude/skills/<name> pull`. The dev repo is where you work; the
`~/.claude/skills/` clone is what runs. (Even simpler if your OS/setup allows:
symlink `~/.claude/skills/<name>` → the Pessoal repo, and you skip the pull —
but the clone+pull model above is the safe, portable default.)

Why plain skill over plugin: no namespace prefix, no `.claude-plugin/` manifest,
no marketplace plumbing, instant `/name` invocation, and `~/.claude/skills/`
live-reloads `SKILL.md` edits within the session. Reach for a **plugin** only
when you need to *distribute* to other people/teams, or bundle agents/hooks/MCP
servers — then see `references/publishing.md` for plugin.json, marketplace.json,
and the install cycle.

## Reference files

- `references/anatomy.md` — directory layout, progressive disclosure, what goes where
- `references/frontmatter.md` — every frontmatter field, command naming, `${CLAUDE_SKILL_DIR}`, `$ARGUMENTS`, dynamic `!` injection
- `references/scripts-pattern.md` — when/how to bundle scripts, calling them, the self-contained-then-DRY approach
- `references/multi-os.md` — Windows/Mac/Linux portability snippets
- `references/secrets-and-config.md` — the env-var-OR-`.env` pattern for user keys, configurable via chat
- `references/testing.md` — validating + iterating on a skill
- `references/publishing.md` — plugin.json, marketplace.json, repo layout, GitHub publish, install/update cycle

Read the one for the step you're on — don't load them all up front.
