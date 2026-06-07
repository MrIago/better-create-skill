# Frontmatter, command names, and substitutions

## Minimal valid frontmatter

```yaml
---
name: my-skill
description: What it does AND when to use it. Put the key use case first.
---
```

Only `description` is strictly recommended; `name` defaults to the directory.
**Validate** against these rules (the official validator enforces them):

- Allowed keys: `name`, `description`, `license`, `allowed-tools`, `metadata`, `compatibility`. Anything else fails validation.
- `name`: kebab-case (`[a-z0-9-]`), ≤ 64 chars, no leading/trailing/double hyphen.
- `description`: ≤ 1024 chars, **no `<` or `>`** (angle brackets break it).
- **No `: ` (colon-space) inside an unquoted description** — YAML reads it as a
  new mapping and the parse fails. Use a dash/parens instead, or quote the whole
  value. (Easy trap: "the lifecycle it skips: the mechanism" breaks; "…skips —
  the mechanism" is fine.)
- The combined description+when_to_use shown in the listing is truncated at 1536 chars — front-load the trigger.

Quick validate (the Anthropic skill-creator ships this script):
```bash
python3 .../skill-creator/scripts/quick_validate.py <skill-dir>
```
Or just check the rules above by hand.

## Writing a description that actually triggers

Models **under-trigger** skills. Be a little pushy: state what it does AND
concrete trigger phrases/contexts, including cases where the user won't name it.

Bad: `Summarize videos.`
Good: `Summarize and analyze any video the user links. Use whenever the user
pastes a YouTube/Instagram URL or asks to watch, study, or pull quotes from a
video — even if they don't say "summarize".`

## Full field reference (Claude Code extends the open standard)

| Field | Use |
|-------|-----|
| `name` | display name / command (see below) |
| `description` | the trigger — what + when |
| `when_to_use` | extra trigger context, appended to description (counts to the 1536 cap) |
| `argument-hint` | autocomplete hint, e.g. `[url] [question]` |
| `arguments` | named positional args for `$name` substitution |
| `disable-model-invocation: true` | only the user can run it (`/name`) — for side-effectful actions (deploy, commit) |
| `user-invocable: false` | only the model can run it — for background knowledge |
| `allowed-tools` | tools pre-approved while active (space/comma list or YAML list) |
| `disallowed-tools` | tools removed while active (e.g. AskUserQuestion for an autonomous loop) |
| `model` / `effort` | override model/effort for the turn |
| `context: fork` + `agent` | run the skill in an isolated subagent |
| `paths` | globs that auto-activate the skill when working on matching files |

## How the command name is decided

- `skills/<dir>/SKILL.md` → command is the **directory name** (`/dir`).
- Plugin `skills/<dir>/SKILL.md` → namespaced: `/plugin:dir`.
- **Plugin-root `SKILL.md`** (no skill dir) → command comes from the frontmatter
  `name` (directory name as fallback). This is the layout to use when the repo
  *is* the single skill — set `name:` to the command you want.

## String substitutions (available in the SKILL.md body)

| Token | Expands to |
|-------|-----------|
| `${CLAUDE_SKILL_DIR}` | the skill's own directory — **use this to call bundled scripts** regardless of cwd: `python3 "${CLAUDE_SKILL_DIR}/scripts/x.py"` |
| `$ARGUMENTS` | everything the user passed after the command |
| `$0`, `$1`, … / `$ARGUMENTS[N]` | positional args (shell-style quoting) |
| `$name` | named arg declared in `arguments:` |
| `${CLAUDE_SESSION_ID}` | current session id (logging, temp names) |

## Dynamic context injection — `` !`cmd` ``

A line starting with `` !`command` `` runs the command **before** the skill
reaches the model; its output replaces the placeholder. Use to inline live data:

```markdown
## Current changes
!`git diff HEAD`
```

This is preprocessing — the model only sees the result, not the command. The `!`
must be at line start or after whitespace. For multi-line, use a fenced ` ```! `
block. (Can be disabled org-wide via `disableSkillShellExecution`.)

## Content lifecycle gotcha

Once invoked, the rendered SKILL.md enters the conversation **once** and stays;
the file is not re-read on later turns. Write standing instructions, not
one-time steps. After auto-compaction, only the most recent ~5k tokens of each
invoked skill are carried forward (25k combined budget) — re-invoke a big skill
if it stops influencing behavior.
